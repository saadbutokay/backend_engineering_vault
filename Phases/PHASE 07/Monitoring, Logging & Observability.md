
```
You've built and deployed your backend.
Now the real work begins: keeping it healthy.

Without observability:
  "The app is slow" — where? why? since when?
  "Users are getting errors" — which errors? how many?
  "The server crashed" — what happened right before?
  You're flying blind. Guessing. Reacting after damage.

With observability:
  "API response time increased 200ms at 14:32" — specific!
  "Error rate jumped from 0.1% to 5% on /checkout" — actionable!
  "Memory leaked 50MB/hour for 6 hours before crash" — preventable!
  You see problems BEFORE users report them.
  You fix root causes, not symptoms.

The THREE PILLARS of observability:
┌─────────────────────────────────────────────────────────────┐
│  METRICS:    Numbers over time                              │
│              CPU 45%, 200 req/s, 45ms avg response         │
│              Tool: Prometheus + Grafana                     │
│                                                             │
│  LOGS:       Timestamped events                             │
│              "User 42 failed login at 14:32:01"             │
│              Tool: structlog + CloudWatch/ELK               │
│                                                             │
│  TRACES:     Request journey across services                │
│              /checkout → auth-service → payment-service     │
│              Tool: OpenTelemetry + Jaeger                   │
└─────────────────────────────────────────────────────────────┘

The FOUR GOLDEN SIGNALS (what to always monitor):
  1. Latency:    How long requests take
  2. Traffic:    How many requests per second
  3. Errors:     Rate of failed requests
  4. Saturation: How full your resources are (CPU, memory, disk)
```

---

## Setup

Bash

```
cd ~/projects
mkdir observability_demo
cd observability_demo
python3 -m venv venv
source venv/bin/activate

pip install fastapi uvicorn prometheus-client \
    structlog loguru opentelemetry-sdk \
    opentelemetry-instrumentation-fastapi \
    opentelemetry-instrumentation-sqlalchemy \
    opentelemetry-exporter-otlp \
    sentry-sdk httpx

touch app.py docker-compose.monitoring.yml
code .
```

---

## 1. 📊 Prometheus — Metrics Collection

### What is Prometheus?

text

```
Prometheus = time-series metrics database.

It works by SCRAPING:
  Every 15 seconds, Prometheus visits your app at /metrics
  Your app returns all current metric values
  Prometheus stores them with timestamp

Then you query:
  "What was my P95 response time over the last hour?"
  "How many 500 errors in the last 5 minutes?"
  "Which endpoint is slowest?"

Prometheus metric types:
  Counter:   only goes up (request count, error count)
  Gauge:     goes up and down (current connections, memory)
  Histogram: distribution of values (response times)
  Summary:   similar to histogram (less flexible)
```

### FastAPI + Prometheus

Python

```
# app/metrics.py
"""
Prometheus metrics for FastAPI.
Exposes metrics at /metrics endpoint.
Prometheus scrapes this endpoint every 15 seconds.
"""
import time
from functools import wraps
from typing import Callable

from prometheus_client import (
    Counter,
    Gauge,
    Histogram,
    Summary,
    Info,
    generate_latest,
    CONTENT_TYPE_LATEST,
    CollectorRegistry,
    multiprocess,
    REGISTRY,
)
from fastapi import FastAPI, Request, Response
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.routing import Match


# ─────────────────────────────────────────
# METRIC DEFINITIONS
# ─────────────────────────────────────────

# Counter: total HTTP requests (never decreases)
http_requests_total = Counter(
    name="http_requests_total",
    documentation="Total number of HTTP requests",
    labelnames=["method", "endpoint", "status_code"],
)

# Histogram: request duration distribution
http_request_duration_seconds = Histogram(
    name="http_request_duration_seconds",
    documentation="HTTP request duration in seconds",
    labelnames=["method", "endpoint"],
    # Buckets: boundaries for the distribution
    # Good for web APIs: 10ms, 25ms, 50ms, 100ms, 250ms, 500ms, 1s, 2.5s
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0],
)

# Gauge: current active requests
http_requests_in_progress = Gauge(
    name="http_requests_in_progress",
    documentation="Number of HTTP requests currently being processed",
    labelnames=["method", "endpoint"],
)

# Counter: errors by type
app_errors_total = Counter(
    name="app_errors_total",
    documentation="Total number of application errors",
    labelnames=["error_type", "endpoint"],
)

# Gauge: database connection pool stats
db_pool_connections = Gauge(
    name="db_pool_connections",
    documentation="Database connection pool size",
    labelnames=["state"],   # 'active', 'idle', 'total'
)

# Counter: background job processing
celery_tasks_total = Counter(
    name="celery_tasks_total",
    documentation="Total Celery tasks processed",
    labelnames=["task_name", "status"],
)

# Histogram: external API call durations
external_api_duration_seconds = Histogram(
    name="external_api_duration_seconds",
    documentation="External API call duration",
    labelnames=["service", "endpoint"],
    buckets=[0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0],
)

# Gauge: cache hit/miss ratio
cache_operations_total = Counter(
    name="cache_operations_total",
    documentation="Cache operations",
    labelnames=["operation", "result"],  # operation: get/set, result: hit/miss
)

# Application info (version, environment)
app_info = Info(
    name="app",
    documentation="Application information",
)
app_info.info({
    "version": "1.0.0",
    "environment": "production",
    "python_version": "3.12",
})


# ─────────────────────────────────────────
# PROMETHEUS MIDDLEWARE
# ─────────────────────────────────────────
class PrometheusMiddleware(BaseHTTPMiddleware):
    """
    Middleware that automatically tracks HTTP metrics.
    Wraps every request to record timing, status, errors.
    """

    def __init__(self, app: FastAPI, app_name: str = "app"):
        super().__init__(app)
        self.app_name = app_name
        self.kwargs = {}

    @staticmethod
    def _get_route_template(request: Request) -> str:
        """
        Get the route template (not the actual URL).

        /users/42/posts → /users/{user_id}/posts
        /posts/my-first-post → /posts/{slug}

        Without this, each unique URL would be a separate metric.
        With it, all user requests collapse to /users/{user_id}.
        """
        for route in request.app.routes:
            match, _ = route.matches(request.scope)
            if match == Match.FULL:
                return route.path
        return request.url.path

    async def dispatch(
        self,
        request: Request,
        call_next: Callable
    ) -> Response:
        # Get the route template
        route = self._get_route_template(request)
        method = request.method

        # Track in-progress requests
        http_requests_in_progress.labels(
            method=method,
            endpoint=route
        ).inc()

        # Record start time
        start_time = time.perf_counter()

        try:
            # Process the request
            response = await call_next(request)
            status_code = response.status_code

        except Exception as e:
            # Count unhandled exceptions
            status_code = 500
            app_errors_total.labels(
                error_type=type(e).__name__,
                endpoint=route
            ).inc()
            raise

        finally:
            # Always record duration
            duration = time.perf_counter() - start_time

            http_requests_in_progress.labels(
                method=method,
                endpoint=route
            ).dec()

            http_request_duration_seconds.labels(
                method=method,
                endpoint=route
            ).observe(duration)

            http_requests_total.labels(
                method=method,
                endpoint=route,
                status_code=status_code
            ).inc()

        return response


# ─────────────────────────────────────────
# METRICS ENDPOINT
# ─────────────────────────────────────────
def setup_metrics(app: FastAPI) -> None:
    """Add Prometheus middleware and /metrics endpoint to FastAPI."""

    app.add_middleware(PrometheusMiddleware)

    @app.get(
        "/metrics",
        include_in_schema=False,  # don't show in API docs
        response_class=Response,
    )
    async def metrics():
        """Prometheus metrics endpoint — scraped every 15 seconds."""
        return Response(
            content=generate_latest(),
            media_type=CONTENT_TYPE_LATEST
        )


# ─────────────────────────────────────────
# HELPER DECORATORS
# ─────────────────────────────────────────
def track_external_call(service: str, endpoint: str):
    """Decorator to track external API call duration."""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, **kwargs):
            start = time.perf_counter()
            try:
                result = await func(*args, **kwargs)
                return result
            finally:
                duration = time.perf_counter() - start
                external_api_duration_seconds.labels(
                    service=service,
                    endpoint=endpoint
                ).observe(duration)
        return wrapper
    return decorator


def track_cache(func: Callable) -> Callable:
    """Decorator to track cache hit/miss."""
    @wraps(func)
    async def wrapper(key: str, *args, **kwargs):
        result = await func(key, *args, **kwargs)
        operation = func.__name__

        if operation == "get":
            result_type = "hit" if result is not None else "miss"
        else:
            result_type = "success"

        cache_operations_total.labels(
            operation=operation,
            result=result_type
        ).inc()

        return result
    return wrapper
```

### FastAPI App with Metrics

Python

```
# app/main.py
from fastapi import FastAPI, Depends, HTTPException, Request
from contextlib import asynccontextmanager
from app.metrics import (
    setup_metrics,
    db_pool_connections,
    app_errors_total,
    track_external_call,
)
import time
import random
import asyncio


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Setup and teardown."""
    print("Starting up with observability...")
    yield
    print("Shutting down...")


app = FastAPI(
    title="Observable FastAPI App",
    lifespan=lifespan,
)

# Add Prometheus metrics
setup_metrics(app)


# ─────────────────────────────────────────
# ROUTES WITH CUSTOM METRICS
# ─────────────────────────────────────────
@app.get("/api/v1/users")
async def list_users(page: int = 1):
    """Simulate a user listing endpoint."""
    # Simulate variable latency
    await asyncio.sleep(random.uniform(0.01, 0.1))
    return {"users": [], "page": page, "total": 1000}


@app.get("/api/v1/users/{user_id}")
async def get_user(user_id: int):
    """Simulate user retrieval."""
    if user_id <= 0:
        app_errors_total.labels(
            error_type="ValidationError",
            endpoint="/api/v1/users/{user_id}"
        ).inc()
        raise HTTPException(422, "Invalid user ID")

    await asyncio.sleep(random.uniform(0.005, 0.05))
    return {"id": user_id, "name": f"User {user_id}"}


@app.post("/api/v1/users")
async def create_user(data: dict):
    """Simulate user creation."""
    await asyncio.sleep(random.uniform(0.05, 0.2))
    return {"id": random.randint(1, 10000), **data}


@app.get("/api/v1/posts")
async def list_posts():
    """Simulate posts listing."""
    await asyncio.sleep(random.uniform(0.02, 0.15))
    return {"posts": []}


@app.post("/api/v1/checkout")
async def checkout(data: dict):
    """
    Checkout with external payment API call.
    Shows how to track external service calls.
    """
    # Simulate payment processing (external call)
    @track_external_call("stripe", "create_payment_intent")
    async def call_stripe():
        await asyncio.sleep(random.uniform(0.1, 0.5))
        if random.random() < 0.05:  # 5% failure rate
            raise Exception("Stripe error")
        return {"payment_id": "pi_abc123"}

    try:
        payment = await call_stripe()
        return {"order_id": "ord_123", "payment": payment}
    except Exception as e:
        app_errors_total.labels(
            error_type="PaymentError",
            endpoint="/api/v1/checkout"
        ).inc()
        raise HTTPException(502, f"Payment failed: {e}")


@app.get("/health")
async def health():
    return {"status": "healthy"}


# ─────────────────────────────────────────
# SIMULATE DB POOL METRICS
# ─────────────────────────────────────────
async def update_db_pool_metrics():
    """Simulate database pool monitoring."""
    while True:
        # In real app: get from SQLAlchemy pool
        active = random.randint(1, 8)
        idle = 10 - active
        db_pool_connections.labels(state="active").set(active)
        db_pool_connections.labels(state="idle").set(idle)
        db_pool_connections.labels(state="total").set(10)
        await asyncio.sleep(15)
```

---

## 2. 📈 Grafana — Visualization

### Setting Up the Monitoring Stack

YAML

```
# docker-compose.monitoring.yml
# Complete monitoring stack:
# Prometheus (metrics DB) + Grafana (dashboards) + Alertmanager (alerts)

version: "3.9"

services:
  # ─── Your App ──────────────────────────────────────────────
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      APP_ENV: production

  # ─── Prometheus (metrics collection) ──────────────────────
  prometheus:
    image: prom/prometheus:v2.49.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./monitoring/alerts.yml:/etc/prometheus/alerts.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'          # allow hot reload
      - '--web.enable-admin-api'
    restart: unless-stopped

  # ─── Grafana (dashboards) ─────────────────────────────────
  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin_password
      GF_USERS_ALLOW_SIGN_UP: "false"
      GF_SERVER_DOMAIN: localhost
      GF_SMTP_ENABLED: "false"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning:ro
      - ./monitoring/grafana/dashboards:/var/lib/grafana/dashboards:ro
    depends_on:
      - prometheus
    restart: unless-stopped

  # ─── Alertmanager (alert routing) ─────────────────────────
  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    restart: unless-stopped

  # ─── Node Exporter (server metrics) ───────────────────────
  node-exporter:
    image: prom/node-exporter:v1.7.0
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped

  # ─── cAdvisor (Docker container metrics) ──────────────────
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.2
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    restart: unless-stopped

  # ─── PostgreSQL Exporter ───────────────────────────────────
  postgres-exporter:
    image: prometheuscommunity/postgres-exporter:v0.15.0
    container_name: postgres-exporter
    ports:
      - "9187:9187"
    environment:
      DATA_SOURCE_NAME: "postgresql://myapp:password@db:5432/myapp_db?sslmode=disable"
    restart: unless-stopped

  # ─── Redis Exporter ───────────────────────────────────────
  redis-exporter:
    image: oliver006/redis_exporter:v1.55.0
    container_name: redis-exporter
    ports:
      - "9121:9121"
    environment:
      REDIS_ADDR: "redis://redis:6379"
      REDIS_PASSWORD: "redis_password"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

### Prometheus Configuration

YAML

```
# monitoring/prometheus.yml
# What Prometheus scrapes and how often

global:
  scrape_interval: 15s          # scrape every 15 seconds
  evaluation_interval: 15s      # evaluate alerts every 15 seconds
  scrape_timeout: 10s

# Alert rules
rule_files:
  - "alerts.yml"

# Alertmanager connection
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

# What to scrape
scrape_configs:
  # Your FastAPI application
  - job_name: "myapp"
    static_configs:
      - targets: ["app:8000"]
    metrics_path: "/metrics"
    scrape_interval: 15s

  # Server metrics (CPU, memory, disk)
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  # Docker container metrics
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]

  # PostgreSQL metrics
  - job_name: "postgresql"
    static_configs:
      - targets: ["postgres-exporter:9187"]

  # Redis metrics
  - job_name: "redis"
    static_configs:
      - targets: ["redis-exporter:9121"]

  # Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # On Kubernetes: use service discovery
  # - job_name: "kubernetes-pods"
  #   kubernetes_sd_configs:
  #     - role: pod
  #   relabel_configs:
  #     - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
  #       action: keep
  #       regex: "true"
```

### Alert Rules

YAML

```
# monitoring/alerts.yml
# PromQL alert rules — when to fire alerts

groups:
  - name: myapp_alerts
    interval: 30s           # evaluate every 30s

    rules:
      # ─── Error Rate ─────────────────────────────────────────
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
          > 0.05
        for: 2m              # must be true for 2 minutes
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate detected"
          description: |
            Error rate is {{ $value | humanizePercentage }} 
            (threshold: 5%)
            This has been ongoing for 2 minutes.

      # ─── Response Time ──────────────────────────────────────
      - alert: SlowResponseTime
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m]))
            by (le, endpoint)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow response time on {{ $labels.endpoint }}"
          description: |
            P95 response time is {{ $value }}s
            on endpoint {{ $labels.endpoint }}

      # ─── High Traffic ───────────────────────────────────────
      - alert: TrafficSpike
        expr: |
          sum(rate(http_requests_total[1m])) > 1000
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Traffic spike detected"
          description: "Current RPS: {{ $value }}"

      # ─── No Traffic ─────────────────────────────────────────
      - alert: NoTraffic
        expr: sum(rate(http_requests_total[5m])) == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "No traffic detected — app may be down"

      # ─── High Memory ────────────────────────────────────────
      - alert: HighMemoryUsage
        expr: |
          container_memory_usage_bytes{name="myapp"}
          /
          container_spec_memory_limit_bytes{name="myapp"}
          > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory at {{ $value | humanizePercentage }}"

      # ─── Database Connections ───────────────────────────────
      - alert: DatabaseConnectionsHigh
        expr: db_pool_connections{state="active"} > 8
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "Active connections: {{ $value }}"

      # ─── Disk Space ─────────────────────────────────────────
      - alert: DiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes{fstype!="tmpfs"}
          /
          node_filesystem_size_bytes{fstype!="tmpfs"})
          < 0.15
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space"
          description: "Only {{ $value | humanizePercentage }} free"

  - name: kubernetes_alerts
    rules:
      # ─── Pod Crash Looping ──────────────────────────────────
      - alert: PodCrashLooping
        expr: |
          rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod crash looping: {{ $labels.pod }}"

      # ─── Deployment Replicas ────────────────────────────────
      - alert: DeploymentReplicasMismatch
        expr: |
          kube_deployment_spec_replicas
          != kube_deployment_status_available_replicas
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Deployment {{ $labels.deployment }} has fewer replicas than desired"
```

### Alertmanager Configuration

YAML

```
# monitoring/alertmanager.yml
# Where to send alerts

global:
  resolve_timeout: 5m
  slack_api_url: "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

# ─── Default route ──────────────────────────────────────────
route:
  receiver: "slack-default"
  group_by: ["alertname", "severity"]
  group_wait: 30s              # wait before first alert
  group_interval: 5m           # wait between group alerts
  repeat_interval: 4h          # resend after 4 hours if still firing

  routes:
    # Critical alerts go to PagerDuty immediately
    - receiver: "pagerduty"
      match:
        severity: critical
      group_wait: 10s

    # Warnings go to Slack only
    - receiver: "slack-warnings"
      match:
        severity: warning

# ─── Receivers ──────────────────────────────────────────────
receivers:
  - name: "slack-default"
    slack_configs:
      - channel: "#alerts"
        title: "{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}"
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Severity:* {{ .Labels.severity }}
          {{ end }}
        send_resolved: true

  - name: "slack-warnings"
    slack_configs:
      - channel: "#backend-warnings"
        send_resolved: true

  - name: "pagerduty"
    pagerduty_configs:
      - routing_key: "YOUR_PAGERDUTY_KEY"
        description: "{{ .CommonAnnotations.summary }}"
        severity: "{{ .CommonLabels.severity }}"

  - name: "email"
    email_configs:
      - to: "oncall@myapp.com"
        from: "alerts@myapp.com"
        smarthost: "smtp.gmail.com:587"
        auth_username: "alerts@myapp.com"
        auth_password: "email_password"
        require_tls: true

# Silencing (suppress alerts during maintenance)
# Create via Alertmanager UI at localhost:9093
```

### Grafana Dashboard (as code)

JSON

```
// monitoring/grafana/dashboards/api_dashboard.json
// Import this into Grafana for instant dashboard
{
  "title": "MyApp API Dashboard",
  "panels": [
    {
      "title": "Request Rate (RPS)",
      "type": "stat",
      "targets": [{
        "expr": "sum(rate(http_requests_total[5m]))",
        "legendFormat": "Requests/s"
      }]
    },
    {
      "title": "Error Rate",
      "type": "stat",
      "targets": [{
        "expr": "sum(rate(http_requests_total{status_code=~'5..'}[5m])) / sum(rate(http_requests_total[5m])) * 100",
        "legendFormat": "Error %"
      }]
    },
    {
      "title": "P95 Response Time",
      "type": "graph",
      "targets": [{
        "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint))",
        "legendFormat": "{{ endpoint }}"
      }]
    },
    {
      "title": "Active Requests",
      "type": "graph",
      "targets": [{
        "expr": "sum(http_requests_in_progress) by (endpoint)",
        "legendFormat": "{{ endpoint }}"
      }]
    },
    {
      "title": "Request Rate by Status",
      "type": "graph",
      "targets": [{
        "expr": "sum(rate(http_requests_total[5m])) by (status_code)",
        "legendFormat": "{{ status_code }}"
      }]
    },
    {
      "title": "DB Connection Pool",
      "type": "gauge",
      "targets": [{
        "expr": "db_pool_connections{state='active'} / db_pool_connections{state='total'} * 100",
        "legendFormat": "Pool Usage %"
      }]
    }
  ]
}
```

---

## 3. 📝 Structured Logging

### Why Structured Logging?

text

```
Unstructured (bad):
  "User 42 logged in from 192.168.1.1 at 14:32:01"
  Hard to parse. Hard to filter. Hard to aggregate.

Structured JSON (good):
  {
    "timestamp": "2024-01-15T14:32:01Z",
    "level": "INFO",
    "event": "user_login",
    "user_id": 42,
    "ip": "192.168.1.1",
    "duration_ms": 45,
    "request_id": "abc-123"
  }

  Now you can:
  → Filter: all events for user_id=42
  → Alert: when user_id appears in too many failed logins
  → Dashboard: login rate per minute
  → Search: find specific request by request_id
```

Python

```
# app/logging_config.py
"""
Production-ready structured logging setup.
Uses structlog for clean, consistent JSON logs.
"""
import logging
import sys
import time
from typing import Any
import structlog
from structlog.types import EventDict, Processor


# ─────────────────────────────────────────
# CUSTOM PROCESSORS
# ─────────────────────────────────────────
def add_app_context(
    logger: Any,
    method: str,
    event_dict: EventDict
) -> EventDict:
    """Add application context to every log entry."""
    event_dict["app"] = "myapp"
    event_dict["environment"] = "production"
    return event_dict


def add_request_context(
    logger: Any,
    method: str,
    event_dict: EventDict
) -> EventDict:
    """
    Add request context (request_id, user_id) if available.
    Uses contextvars to get request-scoped data.
    """
    from contextvars import ContextVar

    # Get request context if set
    request_id_var: ContextVar[str | None] = ContextVar("request_id", default=None)
    user_id_var: ContextVar[int | None] = ContextVar("user_id", default=None)

    request_id = request_id_var.get()
    user_id = user_id_var.get()

    if request_id:
        event_dict["request_id"] = request_id
    if user_id:
        event_dict["user_id"] = user_id

    return event_dict


def drop_color_message_key(
    logger: Any,
    method: str,
    event_dict: EventDict
) -> EventDict:
    """Remove color_message key added by uvicorn."""
    event_dict.pop("color_message", None)
    return event_dict


# ─────────────────────────────────────────
# LOGGING SETUP
# ─────────────────────────────────────────
def setup_logging(
    log_level: str = "INFO",
    json_logs: bool = True,
) -> None:
    """Configure structured logging for the application."""

    # Shared processors (run for every log entry)
    shared_processors: list[Processor] = [
        structlog.contextvars.merge_contextvars,    # merge context vars
        add_app_context,
        structlog.stdlib.add_log_level,             # add level field
        structlog.stdlib.add_logger_name,           # add logger name
        structlog.processors.TimeStamper(fmt="iso"), # add ISO timestamp
        structlog.processors.StackInfoRenderer(),    # format stack traces
        drop_color_message_key,
    ]

    if json_logs:
        # Production: JSON output (parseable by log aggregators)
        processors = shared_processors + [
            structlog.processors.dict_tracebacks,
            structlog.processors.JSONRenderer(),
        ]
    else:
        # Development: colored, human-readable output
        processors = shared_processors + [
            structlog.dev.ConsoleRenderer(colors=True),
        ]

    structlog.configure(
        processors=processors,
        logger_factory=structlog.stdlib.LoggerFactory(),
        wrapper_class=structlog.stdlib.BoundLogger,
        cache_logger_on_first_use=True,
    )

    # Configure standard library logging to go through structlog
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=getattr(logging, log_level.upper()),
    )

    # Quiet noisy loggers
    logging.getLogger("uvicorn.access").setLevel(logging.WARNING)
    logging.getLogger("sqlalchemy.engine").setLevel(logging.WARNING)
    logging.getLogger("httpx").setLevel(logging.WARNING)


# ─────────────────────────────────────────
# LOGGER USAGE
# ─────────────────────────────────────────
# Get a logger (use module name for context)
logger = structlog.get_logger(__name__)


# ─────────────────────────────────────────
# REQUEST CONTEXT MIDDLEWARE
# ─────────────────────────────────────────
import uuid
from contextvars import ContextVar
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import Request, Response

request_id_var: ContextVar[str] = ContextVar("request_id", default="")
user_id_var: ContextVar[int | None] = ContextVar("user_id", default=None)


class LoggingMiddleware(BaseHTTPMiddleware):
    """Add request context to all logs for the duration of a request."""

    async def dispatch(self, request: Request, call_next):
        # Generate unique request ID
        request_id = str(uuid.uuid4())[:8]

        # Set context for this request (all logs get these fields)
        token_request = request_id_var.set(request_id)
        structlog.contextvars.clear_contextvars()
        structlog.contextvars.bind_contextvars(
            request_id=request_id,
            method=request.method,
            path=request.url.path,
        )

        start = time.perf_counter()

        logger.info(
            "Request started",
            client_ip=request.client.host if request.client else "unknown",
        )

        try:
            response = await call_next(request)

            duration_ms = (time.perf_counter() - start) * 1000
            logger.info(
                "Request completed",
                status_code=response.status_code,
                duration_ms=round(duration_ms, 2),
            )

            # Add request ID to response headers
            response.headers["X-Request-ID"] = request_id
            return response

        except Exception as e:
            duration_ms = (time.perf_counter() - start) * 1000
            logger.exception(
                "Request failed",
                error=str(e),
                duration_ms=round(duration_ms, 2),
            )
            raise
        finally:
            request_id_var.reset(token_request)


# ─────────────────────────────────────────
# USAGE IN SERVICES
# ─────────────────────────────────────────
class UserService:
    """Example of structured logging in a service."""

    def __init__(self):
        # Bind service context to logger
        self.log = structlog.get_logger(__name__).bind(
            service="user_service"
        )

    async def register(self, email: str, name: str) -> dict:
        log = self.log.bind(email=email)  # add email to all logs

        log.info("Registration attempt started")

        try:
            # Check duplicate
            if email == "taken@test.com":
                log.warning("Registration failed: duplicate email")
                raise ValueError("Email already registered")

            # Create user
            user_id = 42
            log.info(
                "User registered successfully",
                user_id=user_id,
                method="email"
            )
            return {"id": user_id, "email": email}

        except ValueError:
            raise
        except Exception as e:
            log.exception(
                "Unexpected registration error",
                error_type=type(e).__name__
            )
            raise


# Sample log output (JSON):
# {
#     "timestamp": "2024-01-15T14:32:01.123Z",
#     "level": "info",
#     "event": "User registered successfully",
#     "app": "myapp",
#     "environment": "production",
#     "logger": "app.services.user_service",
#     "service": "user_service",
#     "request_id": "abc-123",
#     "user_id": 42,
#     "email": "alice@test.com",
#     "method": "email"
# }
```

---

## 4. 🔍 OpenTelemetry — Distributed Tracing

### What is Distributed Tracing?

text

```
Problem in microservices:
  User makes a request to /checkout
  → API Gateway service
  → Auth service
  → Inventory service
  → Payment service
  → Notification service

The request is slow. WHERE is it slow?
Without tracing: no idea. Check each service's logs separately.
With tracing: see the ENTIRE journey in one view.

TRACE: the complete journey of a request
SPAN: one operation within a trace

Example trace:
  ┌─────────────────────────────────────────────────────┐
  │ TRACE: POST /checkout (total: 450ms)                │
  │                                                     │
  │ ├─── auth.verify_token (12ms)                      │
  │ ├─── inventory.check_stock (25ms)                  │
  │ ├─── payment.charge_card (380ms) ← BOTTLENECK!     │
  │ │     └─── stripe.api_call (350ms)                 │
  │ └─── notification.send_email (33ms)                │
  └─────────────────────────────────────────────────────┘
  
  Now you know: fix the Stripe API call!
```

Python

```
# app/tracing.py
"""
OpenTelemetry distributed tracing setup.
Traces flow automatically through your code.
Export to Jaeger, Tempo, or OTLP-compatible backend.
"""
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import (
    BatchSpanProcessor,
    ConsoleSpanExporter,  # good for development
)
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource, SERVICE_NAME
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor
from opentelemetry.propagate import set_global_textmap
from opentelemetry.propagators.b3 import B3MultiFormat
from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator
from opentelemetry.propagators.composite import CompositePropagator


def setup_tracing(
    service_name: str = "myapp",
    environment: str = "production",
    otlp_endpoint: str = "http://jaeger:4317",
    export_to_console: bool = False,
) -> trace.Tracer:
    """
    Configure OpenTelemetry distributed tracing.

    Automatically instruments:
    - FastAPI HTTP requests (creates spans)
    - SQLAlchemy queries (creates spans)
    - HTTPX external calls (creates spans)
    - Redis operations (creates spans)

    All spans are linked to parent trace automatically.
    """

    # Resource: metadata about this service
    resource = Resource.create({
        SERVICE_NAME: service_name,
        "service.version": "1.0.0",
        "deployment.environment": environment,
    })

    # Tracer provider: manages all traces
    provider = TracerProvider(resource=resource)

    # Exporters: where to send traces
    if export_to_console:
        # Development: print to console
        provider.add_span_processor(
            BatchSpanProcessor(ConsoleSpanExporter())
        )
    else:
        # Production: send to Jaeger/Tempo via OTLP
        otlp_exporter = OTLPSpanExporter(
            endpoint=otlp_endpoint,
            insecure=True,  # use TLS in production
        )
        provider.add_span_processor(
            BatchSpanProcessor(
                otlp_exporter,
                max_queue_size=2048,
                max_export_batch_size=512,
                export_timeout_millis=30000,
            )
        )

    # Set as global tracer provider
    trace.set_tracer_provider(provider)

    # Propagation: pass trace context between services via HTTP headers
    # W3C TraceContext (standard) + B3 (Zipkin compatible)
    set_global_textmap(CompositePropagator([
        TraceContextTextMapPropagator(),
        B3MultiFormat(),
    ]))

    # Auto-instrument libraries
    # These add spans automatically — no code changes needed!
    FastAPIInstrumentor().instrument()          # HTTP requests
    SQLAlchemyInstrumentor().instrument()       # DB queries
    HTTPXClientInstrumentor().instrument()      # HTTP client calls
    RedisInstrumentor().instrument()            # Redis calls

    # Return tracer for manual spans
    return trace.get_tracer(service_name)


# ─────────────────────────────────────────
# MANUAL SPANS
# ─────────────────────────────────────────
# Get tracer (do this once per module)
tracer = trace.get_tracer(__name__)


async def process_order(order_data: dict) -> dict:
    """
    Manual tracing for complex operations.
    Creates child spans for each step.
    """
    # Start a new span (child of current trace)
    with tracer.start_as_current_span(
        "process_order",
        attributes={
            "order.user_id": order_data.get("user_id"),
            "order.total": str(order_data.get("total")),
            "order.items_count": len(order_data.get("items", [])),
        }
    ) as span:
        try:
            # Step 1: Validate order
            with tracer.start_as_current_span("validate_order") as validate_span:
                # validation logic
                import asyncio
                await asyncio.sleep(0.01)
                validate_span.set_attribute("validation.result", "passed")

            # Step 2: Reserve inventory
            with tracer.start_as_current_span("reserve_inventory") as inv_span:
                await asyncio.sleep(0.02)
                inv_span.set_attribute("inventory.reserved", True)

            # Step 3: Process payment
            with tracer.start_as_current_span("process_payment") as pay_span:
                pay_span.set_attribute("payment.method", "credit_card")
                await asyncio.sleep(0.1)  # external API call
                pay_span.set_attribute("payment.status", "success")
                pay_span.set_attribute("payment.id", "pi_abc123")

            # Step 4: Send confirmation
            with tracer.start_as_current_span("send_confirmation"):
                await asyncio.sleep(0.03)

            # Add result to parent span
            span.set_attribute("order.status", "completed")
            return {"order_id": "ord_123", "status": "completed"}

        except Exception as e:
            # Record the error on the span
            span.record_exception(e)
            span.set_status(trace.StatusCode.ERROR, str(e))
            raise


# ─────────────────────────────────────────
# TRACE CONTEXT PROPAGATION
# ─────────────────────────────────────────
# When calling other services, propagate the trace
import httpx
from opentelemetry.propagate import inject


async def call_payment_service(amount: float) -> dict:
    """
    Call external payment service and propagate trace context.
    The payment service will see itself as a child of this trace.
    """
    headers = {}
    inject(headers)  # adds trace context headers automatically

    # HTTPXClientInstrumentor also does this automatically
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "http://payment-service/charge",
            json={"amount": amount},
            headers=headers  # trace context propagated!
        )
        return response.json()
```

### Jaeger — Trace Visualization

YAML

```
# Add to docker-compose.monitoring.yml

services:
  # ─── Jaeger (trace visualization) ─────────────────────────
  jaeger:
    image: jaegertracing/all-in-one:1.53
    container_name: jaeger
    ports:
      - "16686:16686"   # Jaeger UI
      - "4317:4317"     # OTLP gRPC
      - "4318:4318"     # OTLP HTTP
      - "9411:9411"     # Zipkin format
    environment:
      COLLECTOR_OTLP_ENABLED: "true"
      SPAN_STORAGE_TYPE: badger
      BADGER_EPHEMERAL: "false"
      BADGER_DIRECTORY_VALUE: /badger/data
      BADGER_DIRECTORY_KEY: /badger/key
    volumes:
      - jaeger_data:/badger
    restart: unless-stopped

volumes:
  jaeger_data:
```

---

## 5. 🐛 Sentry — Error Tracking

Python

```
# app/sentry_setup.py
"""
Sentry: error tracking and performance monitoring.

What Sentry does:
  ✅ Captures unhandled exceptions
  ✅ Shows full stack trace
  ✅ Groups similar errors
  ✅ Shows which release introduced the bug
  ✅ Alerts you on new errors
  ✅ Performance monitoring (transactions)
  ✅ Shows error frequency over time
"""
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration
from sentry_sdk.integrations.sqlalchemy import SqlalchemyIntegration
from sentry_sdk.integrations.redis import RedisIntegration
from sentry_sdk.integrations.celery import CeleryIntegration
from sentry_sdk.integrations.logging import LoggingIntegration
import logging


def setup_sentry(
    dsn: str,
    environment: str = "production",
    release: str = "1.0.0",
    traces_sample_rate: float = 0.1,  # 10% of requests traced
    profiles_sample_rate: float = 0.1,
) -> None:
    """Initialize Sentry SDK."""

    if not dsn:
        print("⚠️ SENTRY_DSN not set — Sentry disabled")
        return

    sentry_sdk.init(
        dsn=dsn,
        environment=environment,
        release=release,

        # Integrations: automatic instrumentation
        integrations=[
            FastApiIntegration(
                transaction_style="endpoint",   # use endpoint name as transaction
            ),
            SqlalchemyIntegration(),
            RedisIntegration(),
            CeleryIntegration(),
            LoggingIntegration(
                level=logging.INFO,             # capture INFO+ logs as breadcrumbs
                event_level=logging.ERROR,      # capture ERROR+ logs as events
            ),
        ],

        # Performance monitoring
        traces_sample_rate=traces_sample_rate,
        profiles_sample_rate=profiles_sample_rate,

        # What NOT to send to Sentry (privacy)
        before_send=filter_sentry_events,

        # Ignore specific exceptions
        ignore_errors=[
            KeyboardInterrupt,
            # 404 errors (expected, not bugs):
            # HTTPException with status_code=404
        ],

        # Set user context from request (for debugging)
        send_default_pii=False,  # don't send passwords/tokens!
    )


def filter_sentry_events(event: dict, hint: dict) -> dict | None:
    """
    Filter or modify events before sending to Sentry.
    Return None to DROP the event.
    Return modified event to send it.
    """
    # Don't send 404 errors to Sentry (expected, not bugs)
    if "exc_info" in hint:
        exc_type, exc_value, _ = hint["exc_info"]
        if hasattr(exc_value, "status_code"):
            if exc_value.status_code == 404:
                return None  # drop this event
            if exc_value.status_code == 401:
                return None  # auth errors are expected

    # Sanitize sensitive data
    if "request" in event:
        headers = event["request"].get("headers", {})
        # Remove authorization header
        headers.pop("authorization", None)
        headers.pop("cookie", None)

    return event


# ─────────────────────────────────────────
# MANUAL SENTRY USAGE
# ─────────────────────────────────────────
def capture_payment_error(error: Exception, user_id: int, amount: float):
    """Manually capture an error with context."""
    with sentry_sdk.push_scope() as scope:
        # Add context to this specific error
        scope.set_user({"id": user_id})
        scope.set_tag("payment.amount", str(amount))
        scope.set_tag("error.type", "payment")
        scope.set_extra("payment_data", {
            "amount": amount,
            "currency": "USD",
        })
        sentry_sdk.capture_exception(error)


def add_user_context(user_id: int, email: str) -> None:
    """Set user context for all subsequent errors in this request."""
    sentry_sdk.set_user({
        "id": user_id,
        "email": email,  # careful: PII!
    })


# ─────────────────────────────────────────
# PERFORMANCE TRANSACTIONS
# ─────────────────────────────────────────
def track_with_sentry(operation_name: str):
    """Decorator to track operation performance in Sentry."""
    def decorator(func):
        async def wrapper(*args, **kwargs):
            with sentry_sdk.start_transaction(
                op="task",
                name=operation_name,
            ):
                return await func(*args, **kwargs)
        return wrapper
    return decorator


@track_with_sentry("generate_monthly_report")
async def generate_monthly_report(month: str) -> dict:
    """Generate monthly report — tracked in Sentry Performance."""
    import asyncio
    await asyncio.sleep(5)  # heavy computation
    return {"report": "data"}
```

---

## 6. 🔗 Complete Observability Setup

Python

```
# app/main.py — with full observability

from fastapi import FastAPI
from contextlib import asynccontextmanager
import os
import structlog

from app.logging_config import setup_logging, LoggingMiddleware
from app.metrics import setup_metrics
from app.tracing import setup_tracing
from app.sentry_setup import setup_sentry

logger = structlog.get_logger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application startup with observability."""
    logger.info(
        "Application starting",
        version="1.0.0",
        environment=os.getenv("APP_ENV", "production"),
    )
    yield
    logger.info("Application shutting down")


def create_app() -> FastAPI:
    """Create FastAPI application with full observability."""

    # 1. Setup logging FIRST (other components log)
    setup_logging(
        log_level=os.getenv("LOG_LEVEL", "INFO"),
        json_logs=os.getenv("APP_ENV") == "production",
    )

    # 2. Setup Sentry (catches errors early)
    setup_sentry(
        dsn=os.getenv("SENTRY_DSN", ""),
        environment=os.getenv("APP_ENV", "production"),
        release=os.getenv("APP_VERSION", "1.0.0"),
    )

    # 3. Setup distributed tracing
    tracer = setup_tracing(
        service_name="myapp",
        environment=os.getenv("APP_ENV", "production"),
        otlp_endpoint=os.getenv("OTLP_ENDPOINT", "http://jaeger:4317"),
        export_to_console=os.getenv("APP_ENV") == "development",
    )

    # 4. Create FastAPI app
    app = FastAPI(
        title="Observable FastAPI",
        version="1.0.0",
        lifespan=lifespan,
    )

    # 5. Add observability middleware (ORDER MATTERS!)
    app.add_middleware(LoggingMiddleware)   # logs every request
    setup_metrics(app)                      # prometheus metrics

    # Your routers and business logic
    from app.routers import users, posts, auth
    app.include_router(auth.router, prefix="/api/v1/auth")
    app.include_router(users.router, prefix="/api/v1/users")
    app.include_router(posts.router, prefix="/api/v1/posts")

    return app


app = create_app()
```

---

## 7. ☁️ CloudWatch in Production

Python

```
# app/cloudwatch_logging.py
"""
Send logs to AWS CloudWatch for centralized log management.
Works seamlessly with ECS, EC2, Lambda.
"""
import logging
import json
import boto3
from pythonjsonlogger import jsonlogger
from datetime import datetime


class CloudWatchHandler(logging.Handler):
    """
    Send logs to CloudWatch Logs.
    On ECS/EC2: use awslogs log driver instead (simpler).
    Use this for custom log groups or non-ECS deployments.
    """

    def __init__(self, log_group: str, log_stream: str, region: str = "us-east-1"):
        super().__init__()
        self.log_group = log_group
        self.log_stream = log_stream
        self.client = boto3.client("logs", region_name=region)
        self._setup_log_group()
        self.sequence_token = None

    def _setup_log_group(self) -> None:
        """Create log group and stream if they don't exist."""
        try:
            self.client.create_log_group(logGroupName=self.log_group)
            self.client.put_retention_policy(
                logGroupName=self.log_group,
                retentionInDays=30
            )
        except self.client.exceptions.ResourceAlreadyExistsException:
            pass

        try:
            self.client.create_log_stream(
                logGroupName=self.log_group,
                logStreamName=self.log_stream
            )
        except self.client.exceptions.ResourceAlreadyExistsException:
            pass

    def emit(self, record: logging.LogRecord) -> None:
        """Send log record to CloudWatch."""
        log_entry = self.format(record)

        params = {
            "logGroupName": self.log_group,
            "logStreamName": self.log_stream,
            "logEvents": [{
                "timestamp": int(record.created * 1000),
                "message": log_entry,
            }]
        }

        if self.sequence_token:
            params["sequenceToken"] = self.sequence_token

        try:
            response = self.client.put_log_events(**params)
            self.sequence_token = response["nextSequenceToken"]
        except Exception as e:
            print(f"CloudWatch log error: {e}")


# ─────────────────────────────────────────
# ECS RECOMMENDED APPROACH
# ─────────────────────────────────────────
# In ECS task definition, use awslogs driver:
# "logConfiguration": {
#     "logDriver": "awslogs",
#     "options": {
#         "awslogs-group": "/ecs/myapp",
#         "awslogs-region": "us-east-1",
#         "awslogs-stream-prefix": "ecs",
#         "awslogs-create-group": "true"
#     }
# }
#
# Then your app just prints to stdout,
# ECS sends it to CloudWatch automatically!
#
# In CloudWatch Logs Insights, query logs like SQL:
#
# fields @timestamp, @message
# | filter level = "ERROR"
# | filter service = "user_service"
# | sort @timestamp desc
# | limit 20
#
# Count errors by type:
# fields @timestamp, error_type
# | filter level = "ERROR"
# | stats count(*) as error_count by error_type
# | sort error_count desc
```

---

## 8. 📊 The Four Golden Signals Dashboard

Python

```
# monitoring/golden_signals.py
"""
The Four Golden Signals for every backend service.
Add these PromQL queries to Grafana.
"""

GOLDEN_SIGNALS_PROMQL = {

    # ─── 1. LATENCY ─────────────────────────────────────────
    # How long requests take (use percentiles, not averages!)
    "latency_p50": """
        histogram_quantile(0.50,
            sum(rate(http_request_duration_seconds_bucket[5m]))
            by (le, endpoint)
        ) * 1000
    """,
    # Unit: ms, Target: < 100ms

    "latency_p95": """
        histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m]))
            by (le, endpoint)
        ) * 1000
    """,
    # Unit: ms, Target: < 200ms

    "latency_p99": """
        histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m]))
            by (le)
        ) * 1000
    """,
    # Unit: ms, Target: < 500ms

    # ─── 2. TRAFFIC ──────────────────────────────────────────
    # How many requests per second
    "requests_per_second": """
        sum(rate(http_requests_total[5m]))
    """,
    # Unit: req/s

    "requests_by_endpoint": """
        sum(rate(http_requests_total[5m])) by (endpoint)
    """,

    # ─── 3. ERRORS ───────────────────────────────────────────
    # Rate of failed requests
    "error_rate_percent": """
        sum(rate(http_requests_total{status_code=~"5.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
        * 100
    """,
    # Target: < 0.1%

    "error_rate_by_endpoint": """
        sum(rate(http_requests_total{status_code=~"5.."}[5m])) by (endpoint)
        /
        sum(rate(http_requests_total[5m])) by (endpoint)
        * 100
    """,

    "4xx_rate": """
        sum(rate(http_requests_total{status_code=~"4.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
        * 100
    """,

    # ─── 4. SATURATION ───────────────────────────────────────
    # How full your resources are
    "cpu_usage_percent": """
        100 - (avg by (instance) (
            irate(node_cpu_seconds_total{mode="idle"}[5m])
        ) * 100)
    """,
    # Target: < 80%

    "memory_usage_percent": """
        (
            node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
        )
        /
        node_memory_MemTotal_bytes
        * 100
    """,
    # Target: < 80%

    "disk_usage_percent": """
        (
            node_filesystem_size_bytes - node_filesystem_avail_bytes
        )
        /
        node_filesystem_size_bytes
        * 100
    """,
    # Target: < 85%

    "db_connections_percent": """
        db_pool_connections{state="active"}
        /
        db_pool_connections{state="total"}
        * 100
    """,
    # Target: < 80%

    # ─── BONUS: SLO Tracking ─────────────────────────────────
    # SLO = Service Level Objective (your performance promise)
    # SLA = Service Level Agreement (contractual SLO)
    # SLI = Service Level Indicator (actual measurement)

    # Example SLO: 99.9% of requests < 500ms
    "slo_latency_compliance": """
        sum(rate(http_request_duration_seconds_bucket{le="0.5"}[30d]))
        /
        sum(rate(http_request_duration_seconds_count[30d]))
        * 100
    """,
    # Target: > 99.9%

    # Error budget: how much "budget" until you breach SLO
    "error_budget_remaining": """
        1 - (
            sum(rate(http_requests_total{status_code=~"5.."}[30d]))
            /
            sum(rate(http_requests_total[30d]))
        ) / 0.001  # 0.1% error budget (for 99.9% availability SLO)
        * 100
    """,
}
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│              MONITORING & OBSERVABILITY STACK                  │
│                                                                │
│  THREE PILLARS:                                                │
│  Metrics  → Prometheus (collect) + Grafana (visualize)        │
│  Logs     → structlog (structured) + CloudWatch (store)       │
│  Traces   → OpenTelemetry (instrument) + Jaeger (visualize)  │
│                                                                │
│  ERROR TRACKING:                                               │
│  Sentry → catches exceptions, groups errors, alerts you       │
│                                                                │
│  FOUR GOLDEN SIGNALS:                                          │
│  Latency:    P50, P95, P99 response time                      │
│  Traffic:    Requests per second                              │
│  Errors:     % of 5xx responses                               │
│  Saturation: CPU%, Memory%, DB connections%                   │
│                                                                │
│  PROMETHEUS METRIC TYPES:                                      │
│  Counter:   requests_total (always increases)                 │
│  Gauge:     active_connections (up and down)                  │
│  Histogram: request_duration (distribution/percentiles)       │
│                                                                │
│  KEY MIDDLEWARE:                                               │
│  PrometheusMiddleware → tracks every HTTP request             │
│  LoggingMiddleware    → structured logs with request_id       │
│  OpenTelemetry        → auto-instruments everything           │
│                                                                │
│  ALERTS TO SET UP:                                             │
│  Error rate > 1%       → PagerDuty (critical)                │
│  P95 > 500ms           → Slack (warning)                     │
│  CPU > 80%             → Slack (warning)                     │
│  No traffic for 5min   → PagerDuty (critical)                │
│  Disk < 15% free       → Slack (warning)                     │
│  Pod crash looping     → PagerDuty (critical)                │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What are the three pillars of observability?
2.  What are the four golden signals? Why these four?
3.  What is the difference between Counter, Gauge,
    and Histogram in Prometheus?
4.  Why use P95/P99 instead of average response time?
5.  What is a trace and what is a span?
6.  How does distributed tracing help debug microservices?
7.  What is structured logging and why is it better?
8.  What does structlog.contextvars do?
9.  What is Sentry and what does it capture?
10. What is an alerting rule in Prometheus?
11. What is an SLO? SLA? SLI?
12. What does PrometheusMiddleware do?
13. How does OpenTelemetry auto-instrumentation work?
14. What is the /metrics endpoint and who scrapes it?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Prometheus + Grafana Setup
# a) Start the monitoring docker-compose stack
# b) Run your FastAPI app with PrometheusMiddleware
# c) Generate traffic: for i in {1..100}; do curl http://localhost:8000/api/v1/posts; done
# d) Open Grafana (localhost:3000)
# e) Add Prometheus as data source
# f) Create a dashboard with:
#    - Request rate (RPS) stat panel
#    - Error rate stat panel
#    - P95 response time graph
#    - Requests by endpoint bar chart
# g) Import a pre-built dashboard (ID: 17658 - FastAPI)

# Exercise 2: Alert Rules
# Write Prometheus alert rules for:
# a) Error rate > 5% for 2 minutes → critical
# b) P95 response > 1 second for 5 minutes → warning
# c) Any endpoint getting 0 requests for 10 minutes → warning
# d) More than 5 Celery task failures in 5 minutes → critical
# Test each alert by deliberately triggering it

# Exercise 3: Structured Logging
# Add structlog to your FastAPI project:
# a) Setup JSON logging in production
# b) Add LoggingMiddleware (request_id on every log)
# c) In UserService: log all operations with relevant context
#    (user_id, action, duration, result)
# d) Add correlation ID across services
# e) Query your logs: find all requests for user_id=42

# Exercise 4: Error Tracking with Sentry
# a) Create free Sentry account (sentry.io)
# b) Create new project (FastAPI)
# c) Add Sentry to your app
# d) Trigger an unhandled exception
# e) Find it in Sentry dashboard
# f) Add user context to errors
# g) Create alert: email on first occurrence of new error

# Exercise 5: Distributed Tracing
# a) Start Jaeger with docker-compose
# b) Add OpenTelemetry to your FastAPI app
# c) Make a request that:
#    - Queries the database (SQLAlchemy span)
#    - Calls external API (httpx span)
#    - Uses Redis cache (redis span)
# d) Find the trace in Jaeger UI (localhost:16686)
# e) Identify the slowest component
# f) Add a custom span for your business logic
```

---

## 🎉 Phase 7 Complete!

text

```
✅ 7.1 — Linux & Server Administration
         File system, processes, Nginx, systemd, SSH, firewall

✅ 7.2 — Docker & Containerization
         Dockerfile, multi-stage, docker-compose, production setup

✅ 7.3 — CI/CD Pipeline
         GitHub Actions, automated testing, deployment, strategies

✅ 7.4 — Cloud Services (AWS)
         IAM, EC2, RDS, S3, ElastiCache, ECR, ECS, Lambda, CloudWatch

✅ 7.5 — Kubernetes
         Pods, Deployments, Services, Ingress, HPA, Helm

✅ 7.6 — Monitoring & Observability
         Prometheus, Grafana, structlog, OpenTelemetry, Sentry
```

**You now know the complete DevOps and infrastructure layer.**  
**Your apps can be deployed, scaled, monitored, and maintained.**
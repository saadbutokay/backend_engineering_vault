
```
Docker Compose: great for running containers on ONE server.

The problem at scale:
  Your app grows. One server isn't enough.
  You need 10 servers running your containers.
  
  Questions you face:
  - Which server runs which container?
  - What if a server dies? Who restarts containers?
  - How do you deploy a new version without downtime?
  - How do containers talk to each other across servers?
  - How do you scale from 2 to 20 containers under load?
  - How do you manage secrets across all containers?

Docker Compose: you manage all this manually.
Kubernetes:     handles all of this automatically.

Kubernetes (K8s) is:
  ✅ Container orchestration platform
  ✅ Self-healing (restarts failed containers)
  ✅ Auto-scaling (more containers under load)
  ✅ Rolling deployments (zero downtime)
  ✅ Service discovery (containers find each other)
  ✅ Load balancing (built-in)
  ✅ Secret management
  ✅ Used by: Google, Spotify, GitHub, Reddit, Airbnb

Industry reality:
  If you work at a company with > 10 backend engineers:
  80% chance they use Kubernetes.
```

---

## Setup — Local Kubernetes

Bash

```
# Install kubectl (Kubernetes CLI)
# Mac:
brew install kubectl

# Linux:
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
    https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Windows: Download from kubernetes.io/docs/tasks/tools/install-kubectl-windows

# Install minikube (local Kubernetes cluster)
# Mac:
brew install minikube

# Linux:
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start local cluster
minikube start --cpus=2 --memory=4g --driver=docker

# Verify
kubectl cluster-info
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   1m    v1.29.0

# Install k9s (amazing K8s TUI dashboard)
brew install k9s    # Mac
# Linux: download from k9scli.io

# Create project structure
mkdir -p ~/projects/kubernetes_demo/k8s/{base,overlays/{dev,prod}}
cd ~/projects/kubernetes_demo
```

---

## 1. 🧠 Kubernetes Architecture

text

```
KUBERNETES CLUSTER COMPONENTS:

┌──────────────────────────────────────────────────────────────┐
│                    CONTROL PLANE                             │
│  ┌─────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  │ API Server  │ │ Scheduler│ │Controller│ │  etcd    │    │
│  │(all commands│ │(decides  │ │ Manager  │ │(database │    │
│  │ go through) │ │where pods│ │(ensures  │ │of cluster│    │
│  │             │ │ go)      │ │ desired  │ │ state)   │    │
│  │             │ │          │ │ state)   │ │          │    │
│  └─────────────┘ └──────────┘ └──────────┘ └──────────┘    │
└──────────────────────────────────────────────────────────────┘
         ↕ API calls
┌──────────────────────────────────────────────────────────────┐
│                    WORKER NODES                              │
│                                                              │
│  ┌──────────────────────┐  ┌──────────────────────────────┐ │
│  │   Node 1             │  │   Node 2                     │ │
│  │  ┌─────┐ ┌─────┐    │  │  ┌─────┐ ┌─────┐ ┌─────┐   │ │
│  │  │ Pod │ │ Pod │    │  │  │ Pod │ │ Pod │ │ Pod │   │ │
│  │  └─────┘ └─────┘    │  │  └─────┘ └─────┘ └─────┘   │ │
│  │  kubelet | kube-proxy│  │  kubelet | kube-proxy       │ │
│  └──────────────────────┘  └──────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘

KEY CONCEPTS:
  Node:        A machine (physical or VM) in the cluster
  Pod:         Smallest deployable unit (1+ containers)
  Deployment:  Declares desired state (how many pods, which image)
  Service:     Network access to pods (load balancing)
  Ingress:     External HTTP/HTTPS access (routes to services)
  ConfigMap:   Non-secret configuration
  Secret:      Sensitive data (passwords, keys)
  Namespace:   Virtual cluster (isolate environments)
  Volume:      Persistent storage for pods
  HPA:         Horizontal Pod Autoscaler (scale by CPU/RAM)
```

---

## 2. 📦 Pods — The Smallest Unit

YAML

```
# k8s/base/pod.yaml
# A Pod = one or more containers that share:
#   - Network namespace (same IP address)
#   - Storage volumes
#   - Lifecycle (start/stop together)
#
# In practice: usually one container per pod
# Multi-container: when containers are tightly coupled
#   (e.g., app + sidecar logger)

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    version: "1.0"
    environment: production

spec:
  containers:
    - name: myapp
      image: myapp:latest

      # Resource requests and limits (CRITICAL!)
      # Request: guaranteed minimum (scheduler uses this)
      # Limit:   maximum allowed (OOM killed if exceeded)
      resources:
        requests:
          cpu: "250m"        # 250 millicores = 0.25 CPU
          memory: "256Mi"    # 256 megabytes
        limits:
          cpu: "500m"        # max 0.5 CPU
          memory: "512Mi"    # OOM killed if exceeded

      # Environment variables
      env:
        - name: APP_ENV
          value: "production"
        - name: PORT
          value: "8000"
        # From Secret (encrypted)
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets    # Secret object name
              key: database-url      # key within Secret
        # From ConfigMap
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: log-level

      ports:
        - containerPort: 8000
          protocol: TCP

      # Health checks — Kubernetes uses these to manage pods
      livenessProbe:           # is the container alive?
        httpGet:
          path: /health
          port: 8000
        initialDelaySeconds: 30    # wait 30s before first check
        periodSeconds: 10          # check every 10s
        timeoutSeconds: 5
        failureThreshold: 3        # 3 failures = restart container

      readinessProbe:          # is the container ready for traffic?
        httpGet:
          path: /health
          port: 8000
        initialDelaySeconds: 15
        periodSeconds: 5
        failureThreshold: 3
        # Pod removed from Service endpoints until ready
        # This prevents traffic to warming-up containers

      startupProbe:            # is the app starting? (long startup allowed)
        httpGet:
          path: /health
          port: 8000
        failureThreshold: 30
        periodSeconds: 10      # 30 * 10 = 300s to start

      # Volume mounts
      volumeMounts:
        - name: uploads
          mountPath: /app/uploads
        - name: config
          mountPath: /app/config
          readOnly: true

  # Volumes available to all containers in pod
  volumes:
    - name: uploads
      persistentVolumeClaim:
        claimName: uploads-pvc

    - name: config
      configMap:
        name: myapp-config

  # Which node to schedule on (optional)
  nodeSelector:
    node-type: application

  # Don't run two pods on same node (high availability)
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values: [myapp]
            topologyKey: kubernetes.io/hostname

  # Container restart policy
  restartPolicy: Always
```

Bash

```
# Pod operations
kubectl apply -f k8s/base/pod.yaml         # create/update
kubectl get pods                            # list pods
kubectl get pods -o wide                   # with node info
kubectl describe pod myapp-pod             # detailed info
kubectl logs myapp-pod                     # view logs
kubectl logs myapp-pod -f                  # follow logs
kubectl logs myapp-pod --previous          # crashed pod logs
kubectl exec -it myapp-pod -- bash         # shell into pod
kubectl exec -it myapp-pod -- python manage.py shell
kubectl delete pod myapp-pod               # delete pod
kubectl port-forward pod/myapp-pod 8080:8000  # forward port locally
```

---

## 3. 🔄 Deployment — Desired State

YAML

```
# k8s/base/deployment.yaml
# Deployment = manages a set of identical pods
# Ensures N replicas are always running
# Handles rolling updates and rollbacks

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubernetes.io/change-cause: "Initial deployment v1.0.0"

spec:
  # How many pod replicas to run
  replicas: 3

  # Which pods this Deployment manages
  selector:
    matchLabels:
      app: myapp

  # Rolling update strategy (zero downtime!)
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1           # allow 1 extra pod during update
      maxUnavailable: 0     # never take pods offline during update
      # With 3 replicas: creates 4th pod → verifies → removes old
      # Traffic always served by at least 3 pods

  # Pod template
  template:
    metadata:
      labels:
        app: myapp
        version: "1.0.0"

    spec:
      # Grace period to finish current requests
      terminationGracePeriodSeconds: 30

      # Security context for all containers
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001

      # Pull image from private registry
      imagePullSecrets:
        - name: registry-credentials

      containers:
        - name: myapp
          image: ghcr.io/myorg/myapp:1.0.0
          imagePullPolicy: Always    # always pull latest

          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          env:
            - name: APP_ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: redis-url
            - name: SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: secret-key

          ports:
            - containerPort: 8000

          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3

          # Security hardening
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: [ALL]

          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: uploads
              mountPath: /app/uploads

      volumes:
        - name: tmp
          emptyDir: {}          # in-memory temp storage
        - name: uploads
          persistentVolumeClaim:
            claimName: uploads-pvc

      # Spread pods across availability zones
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: myapp
```

Bash

```
# Deployment operations
kubectl apply -f k8s/base/deployment.yaml

# View deployments
kubectl get deployments
kubectl get deployment myapp -o yaml    # full YAML

# Describe (events, conditions, pod template)
kubectl describe deployment myapp

# Rollout status (watch the rolling update)
kubectl rollout status deployment/myapp

# History
kubectl rollout history deployment/myapp
kubectl rollout history deployment/myapp --revision=2

# Scale manually
kubectl scale deployment myapp --replicas=5

# Update image (triggers rolling update)
kubectl set image deployment/myapp myapp=ghcr.io/myorg/myapp:1.1.0

# Rollback to previous version
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=2

# Pause rolling update (investigate mid-deployment)
kubectl rollout pause deployment/myapp
kubectl rollout resume deployment/myapp

# Restart all pods (force redeploy same image)
kubectl rollout restart deployment/myapp
```

---

## 4. 🌐 Services — Network Access

YAML

```
# k8s/base/service.yaml
# Service = stable network endpoint for a set of pods
# Pods are ephemeral (die and get new IPs)
# Service provides a stable IP and DNS name

# ─────────────────────────────────────────────────
# ClusterIP: internal only (within the cluster)
# ─────────────────────────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
  labels:
    app: myapp

spec:
  type: ClusterIP      # internal access only
  selector:
    app: myapp         # forward to pods with this label

  ports:
    - name: http
      protocol: TCP
      port: 80         # service port (what you connect to)
      targetPort: 8000 # container port (what app listens on)

---
# ─────────────────────────────────────────────────
# NodePort: accessible on each node's IP + port
# Good for development, not production
# ─────────────────────────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport

spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30080    # external port (30000-32767)

---
# ─────────────────────────────────────────────────
# LoadBalancer: creates cloud load balancer
# Works on AWS/GCP/Azure (creates ALB/ELB automatically)
# ─────────────────────────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: myapp-loadbalancer
  annotations:
    # AWS specific: create NLB instead of CLB
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"

spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - name: http
      port: 80
      targetPort: 8000
    - name: https
      port: 443
      targetPort: 8000

---
# ─────────────────────────────────────────────────
# Headless Service: for StatefulSets (databases)
# ─────────────────────────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless

spec:
  clusterIP: None        # headless!
  selector:
    app: postgres
  ports:
    - port: 5432

# ─────────────────────────────────────────────────
# Database service (internal only)
# ─────────────────────────────────────────────────
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service

spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

Bash

```
# Service operations
kubectl get services                    # list all services
kubectl get svc                        # shorthand
kubectl describe service myapp-service
kubectl get endpoints myapp-service    # see which pods are targeted

# Test service connectivity from inside cluster
kubectl run test-pod --image=busybox --rm -it -- \
    wget -qO- http://myapp-service/health

# Port forward service to localhost
kubectl port-forward service/myapp-service 8080:80
```

---

## 5. 🌍 Ingress — External HTTP Access

YAML

```
# k8s/base/ingress.yaml
# Ingress = HTTP/HTTPS routing rules
# One Ingress can route to multiple services
# Works with Nginx Ingress Controller or AWS ALB Ingress Controller

# First: install nginx ingress controller
# kubectl apply -f \
#   https://raw.githubusercontent.com/kubernetes/ingress-nginx/\
#   controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    # Nginx ingress annotations
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "20m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"

    # Rate limiting
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-burst-multiplier: "5"

    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://myapp.com"

    # TLS certificate (cert-manager)
    cert-manager.io/cluster-issuer: "letsencrypt-prod"

spec:
  ingressClassName: nginx

  # TLS configuration
  tls:
    - hosts:
        - myapp.com
        - api.myapp.com
      secretName: myapp-tls-secret   # cert-manager creates this

  rules:
    # Main app
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

    # API subdomain
    - host: api.myapp.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

          - path: /docs
            pathType: Exact
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

    # Separate service for different paths
    - host: myapp.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
          - path: /ws
            pathType: Prefix
            backend:
              service:
                name: myapp-websocket-service
                port:
                  number: 80
```

Bash

```
# Install cert-manager for automatic TLS certificates
kubectl apply -f \
    https://github.com/cert-manager/cert-manager/releases/download/\
    v1.14.4/cert-manager.yaml

# Create Let's Encrypt ClusterIssuer
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@myapp.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
EOF

# View ingress
kubectl get ingress
kubectl describe ingress myapp-ingress
```

---

## 6. 🔐 ConfigMaps & Secrets

YAML

```
# k8s/base/configmap.yaml
# ConfigMap: non-sensitive configuration

apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production

data:
  # Simple key-value pairs
  APP_ENV: "production"
  LOG_LEVEL: "INFO"
  WORKERS: "4"
  ALLOWED_HOSTS: "myapp.com,api.myapp.com"

  # Multi-line values (files)
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://myapp-service;
      }
    }

  app.ini: |
    [database]
    pool_size = 5
    max_overflow = 10

    [cache]
    ttl = 300

---
# k8s/base/secret.yaml
# Secret: sensitive data (base64 encoded — NOT encrypted by default!)
# For real encryption: use Sealed Secrets or AWS Secrets Manager + ESO

apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production

type: Opaque

# Values must be base64 encoded:
# echo -n "myvalue" | base64
data:
  database-url: cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bob3N0OjU0MzIvZGI=
  redis-url: cmVkaXM6Ly86cGFzc0Bob3N0OjYzNzk=
  secret-key: c3VwZXItc2VjcmV0LWtleS0zMi1jaGFycw==
  jwt-secret: and0LXNlY3JldC0zMi1jaGFycw==

# OR use stringData (auto-encodes for you):
stringData:
  stripe-key: "sk_live_your_stripe_key_here"
  sendgrid-key: "SG.your_sendgrid_key_here"

---
# Registry credentials (for pulling private images)
apiVersion: v1
kind: Secret
metadata:
  name: registry-credentials
  namespace: production
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

Bash

```
# ConfigMap operations
kubectl create configmap myapp-config \
    --from-literal=LOG_LEVEL=INFO \
    --from-literal=WORKERS=4 \
    --from-file=nginx.conf=./nginx.conf

kubectl get configmap
kubectl describe configmap myapp-config
kubectl edit configmap myapp-config      # edit in place

# Secret operations
kubectl create secret generic myapp-secrets \
    --from-literal=database-url="postgresql://..." \
    --from-literal=secret-key="your-secret"

# Create registry secret
kubectl create secret docker-registry registry-credentials \
    --docker-server=ghcr.io \
    --docker-username=myusername \
    --docker-password=mytoken \
    --docker-email=admin@myapp.com

# Decode a secret value
kubectl get secret myapp-secrets \
    -o jsonpath='{.data.database-url}' | base64 -d

# Better: use External Secrets Operator (syncs from AWS Secrets Manager)
# ESO pulls secrets from AWS and creates K8s Secrets automatically
```

---

## 7. 📊 HPA — Horizontal Pod Autoscaler

YAML

```
# k8s/base/hpa.yaml
# HPA: automatically scales pods based on CPU/RAM/custom metrics

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp             # which deployment to scale

  minReplicas: 2            # always have at least 2
  maxReplicas: 20           # never exceed 20

  metrics:
    # Scale on CPU usage
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # scale when avg CPU > 70%

    # Scale on memory usage
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

    # Scale on custom metric (requests per second)
    # Requires metrics-server + custom metrics adapter
    - type: Pods
      pods:
        metric:
          name: requests_per_second
        target:
          type: AverageValue
          averageValue: "100"       # 100 req/s per pod

  # Control scaling behavior
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60    # wait 60s before scaling up
      policies:
        - type: Pods
          value: 2                      # add max 2 pods at a time
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300   # wait 5min before scaling down
      policies:
        - type: Percent
          value: 25                     # remove max 25% of pods at a time
          periodSeconds: 60
```

Bash

```
# HPA operations
kubectl get hpa
kubectl describe hpa myapp-hpa

# Watch HPA scaling
kubectl get hpa --watch

# Install metrics-server (required for CPU/Memory HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/\
releases/latest/download/components.yaml

# View resource usage
kubectl top pods
kubectl top nodes
```

---

## 8. 💾 Persistent Volumes

YAML

```
# k8s/base/storage.yaml
# Persistent storage that survives pod restarts

# ─────────────────────────────────────────────────
# StorageClass: defines how storage is provisioned
# ─────────────────────────────────────────────────
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs   # AWS EBS provisioner
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Retain                 # keep volume when PVC deleted
allowVolumeExpansion: true            # allow resizing
volumeBindingMode: WaitForFirstConsumer

---
# ─────────────────────────────────────────────────
# PersistentVolumeClaim: request for storage
# ─────────────────────────────────────────────────
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uploads-pvc
  namespace: production

spec:
  accessModes:
    - ReadWriteOnce        # one pod can write at a time
    # ReadWriteMany: multiple pods can write (requires NFS/EFS)

  storageClassName: fast-ssd

  resources:
    requests:
      storage: 10Gi        # 10 GB

---
# ─────────────────────────────────────────────────
# StatefulSet: for stateful apps (databases)
# Unlike Deployment: pods have stable names + storage
# ─────────────────────────────────────────────────
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis

spec:
  serviceName: redis-headless
  replicas: 1

  selector:
    matchLabels:
      app: redis

  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          command:
            - redis-server
            - --requirepass
            - $(REDIS_PASSWORD)
            - --appendonly
            - "yes"

          env:
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: redis-password

          ports:
            - containerPort: 6379

          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          volumeMounts:
            - name: redis-data
              mountPath: /data

  # VolumeClaimTemplate: each pod gets its own PVC
  volumeClaimTemplates:
    - metadata:
        name: redis-data
      spec:
        accessModes: [ReadWriteOnce]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 5Gi
```

---

## 9. 🏗️ Namespaces — Isolation

YAML

```
# k8s/base/namespaces.yaml
# Namespaces: virtual clusters within a cluster
# Isolate resources by environment, team, or app

apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production

---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging

---
# ResourceQuota: limit total resources per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production

spec:
  hard:
    requests.cpu: "4"          # max 4 CPU cores requested
    requests.memory: 8Gi       # max 8GB RAM requested
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "50"                 # max 50 pods
    services: "20"             # max 20 services
    persistentvolumeclaims: "10"

---
# LimitRange: default resource limits for pods
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production

spec:
  limits:
    - type: Container
      default:              # default limit if not specified
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:       # default request if not specified
        cpu: "250m"
        memory: "256Mi"
      max:                  # maximum allowed
        cpu: "2"
        memory: "2Gi"
      min:                  # minimum allowed
        cpu: "100m"
        memory: "128Mi"
```

Bash

```
# Namespace operations
kubectl get namespaces
kubectl create namespace staging

# Run commands in specific namespace
kubectl get pods -n production
kubectl get pods -n staging
kubectl get pods --all-namespaces    # all namespaces

# Set default namespace (avoid -n every time)
kubectl config set-context --current --namespace=production

# View resource usage per namespace
kubectl describe quota -n production
kubectl describe limitrange -n production
```

---

## 10. 🔄 Complete Application Deployment

YAML

```
# k8s/production/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production

---
# k8s/production/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production
type: Opaque
stringData:
  database-url: "postgresql://myapp:password@rds-endpoint:5432/myapp_db"
  redis-url: "redis://:password@elasticache-endpoint:6379"
  secret-key: "your-32-char-secret-key-here!!"
  jwt-secret-key: "your-32-char-jwt-secret-key!!"

---
# k8s/production/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  APP_ENV: "production"
  LOG_LEVEL: "INFO"
  WORKERS: "4"

---
# k8s/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      imagePullSecrets:
        - name: registry-credentials
      containers:
        - name: myapp
          image: ghcr.io/myorg/myapp:latest
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          envFrom:
            - configMapRef:
                name: myapp-config
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: redis-url
            - name: SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: secret-key
          ports:
            - containerPort: 8000
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 5

---
# k8s/production/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8000

---
# k8s/production/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts: [myapp.com]
      secretName: myapp-tls
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

---
# k8s/production/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Bash

```
# Deploy everything
kubectl apply -f k8s/production/

# Watch the deployment
kubectl rollout status deployment/myapp -n production

# Check everything is running
kubectl get all -n production

# View logs
kubectl logs -f deployment/myapp -n production

# Scale manually
kubectl scale deployment myapp --replicas=5 -n production
```

---

## 11. 🛠️ kubectl — Daily Commands

Bash

```
# ─────────────────────────────────────────
# CONTEXTS (different clusters)
# ─────────────────────────────────────────
kubectl config get-contexts              # list clusters
kubectl config current-context          # current cluster
kubectl config use-context production   # switch cluster
kubectl config use-context minikube     # switch to local

# ─────────────────────────────────────────
# CORE RESOURCES
# ─────────────────────────────────────────
kubectl get all                         # everything in default namespace
kubectl get all -n production           # specific namespace
kubectl get all --all-namespaces        # everywhere

# Short names
kubectl get po                          # pods
kubectl get svc                         # services
kubectl get deploy                      # deployments
kubectl get ing                         # ingresses
kubectl get cm                          # configmaps
kubectl get secret                      # secrets
kubectl get pvc                         # persistent volume claims
kubectl get node                        # nodes

# Output formats
kubectl get pods -o wide                # extra columns
kubectl get pods -o yaml                # full YAML
kubectl get pods -o json                # JSON
kubectl get pods -o jsonpath='{.items[*].metadata.name}'  # extract fields

# Labels and selectors
kubectl get pods -l app=myapp           # filter by label
kubectl get pods -l app=myapp,env=prod  # multiple labels

# ─────────────────────────────────────────
# DEBUGGING
# ─────────────────────────────────────────
kubectl describe pod myapp-abc123       # detailed info + events
kubectl logs myapp-abc123               # container logs
kubectl logs myapp-abc123 -f            # follow
kubectl logs myapp-abc123 --previous    # crashed container
kubectl logs myapp-abc123 -c myapp      # specific container in pod
kubectl logs deployment/myapp           # any pod in deployment

# Execute commands
kubectl exec -it myapp-abc123 -- bash
kubectl exec -it myapp-abc123 -- python -c "import app"
kubectl exec -it myapp-abc123 -c myapp -- bash  # specific container

# Copy files
kubectl cp myapp-abc123:/app/logs/error.log ./error.log
kubectl cp ./config.json myapp-abc123:/app/config.json

# Port forward
kubectl port-forward pod/myapp-abc123 8080:8000
kubectl port-forward service/myapp-service 8080:80
kubectl port-forward deployment/myapp 8080:8000

# ─────────────────────────────────────────
# APPLY / DELETE
# ─────────────────────────────────────────
kubectl apply -f file.yaml              # create or update
kubectl apply -f directory/             # apply all files
kubectl apply -k .                      # kustomize
kubectl delete -f file.yaml            # delete from file
kubectl delete pod myapp-abc123         # delete specific resource
kubectl delete pods -l app=myapp        # delete by label
kubectl delete all -l app=myapp         # delete all resources

# Dry run (see what would happen)
kubectl apply -f deployment.yaml --dry-run=client
kubectl diff -f deployment.yaml         # show what would change

# ─────────────────────────────────────────
# ROLLING UPDATES
# ─────────────────────────────────────────
kubectl set image deployment/myapp myapp=myapp:v2.0.0
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp
kubectl rollout restart deployment/myapp

# ─────────────────────────────────────────
# RESOURCE USAGE
# ─────────────────────────────────────────
kubectl top pods
kubectl top pods -n production
kubectl top nodes

# ─────────────────────────────────────────
# EVENTS
# ─────────────────────────────────────────
kubectl get events --sort-by=.lastTimestamp
kubectl get events -n production --sort-by=.lastTimestamp
kubectl get events --field-selector type=Warning
```

---

## 12. 🔄 CI/CD with Kubernetes

YAML

```
# .github/workflows/deploy-k8s.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      # Build and push Docker image
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: type=sha,prefix=sha-,format=short

      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}

      # Deploy to Kubernetes
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: "latest"

      - name: Configure kubectl
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config
          chmod 600 ~/.kube/config

      - name: Update deployment image
        run: |
          kubectl set image deployment/myapp \
            myapp=ghcr.io/${{ github.repository }}:sha-${{ github.sha }} \
            -n production

      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/myapp \
            -n production \
            --timeout=5m

      - name: Verify deployment
        run: |
          kubectl get pods -n production -l app=myapp
          kubectl get svc -n production

      # On failure: rollback
      - name: Rollback on failure
        if: failure()
        run: |
          kubectl rollout undo deployment/myapp -n production
          echo "⚠️ Rolled back to previous version"
```

---

## 13. 🎯 Helm — Package Manager for Kubernetes

Bash

```
# Helm: package manager for Kubernetes
# Like pip for Python, but for Kubernetes manifests
# A "chart" is a package of Kubernetes templates

# Install Helm
brew install helm    # Mac
# Linux: curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add popular chart repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add cert-manager https://charts.jetstack.io
helm repo update

# Install PostgreSQL (no manual YAML needed!)
helm install myapp-postgres bitnami/postgresql \
    --namespace production \
    --set auth.postgresPassword=mysecretpassword \
    --set auth.database=myapp \
    --set primary.persistence.size=20Gi

# Install Redis
helm install myapp-redis bitnami/redis \
    --namespace production \
    --set auth.password=myredispassword

# Install Nginx Ingress Controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx \
    --create-namespace

# Install cert-manager
helm install cert-manager cert-manager/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --set installCRDs=true

# Create your own Helm chart
helm create myapp
# Creates:
# myapp/
#   Chart.yaml         ← metadata
#   values.yaml        ← default configuration
#   templates/         ← Kubernetes templates
#     deployment.yaml
#     service.yaml
#     ingress.yaml
#     hpa.yaml
#     _helpers.tpl

# Install your chart
helm install myapp ./myapp \
    --namespace production \
    -f values.production.yaml

# values.production.yaml
# image:
#   tag: "1.2.3"
# replicas: 3
# resources:
#   requests:
#     cpu: 250m
#     memory: 256Mi

# Upgrade
helm upgrade myapp ./myapp \
    -f values.production.yaml \
    --namespace production

# Rollback
helm rollback myapp 1 --namespace production

# List releases
helm list -n production

# Check history
helm history myapp -n production
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                    KUBERNETES ESSENTIALS                       │
│                                                                │
│  RESOURCE HIERARCHY:                                           │
│  Namespace → Deployment → ReplicaSet → Pods → Containers     │
│                                                                │
│  KEY RESOURCES:                                                │
│  Pod:          1+ containers, smallest deployable unit        │
│  Deployment:   manages pod replicas + rolling updates         │
│  Service:      stable network endpoint (ClusterIP/LB)         │
│  Ingress:      HTTP/HTTPS routing to services                 │
│  ConfigMap:    non-secret configuration                       │
│  Secret:       sensitive data (passwords, keys)               │
│  PVC:          persistent storage request                     │
│  HPA:          auto-scale by CPU/RAM/custom metrics           │
│  StatefulSet:  ordered, stable pods for databases             │
│  Namespace:    virtual cluster isolation                      │
│                                                                │
│  HEALTH CHECKS:                                                │
│  livenessProbe:  is it alive? (restart if fail)              │
│  readinessProbe: is it ready? (remove from LB if fail)       │
│  startupProbe:   is it starting? (slow start protection)     │
│                                                                │
│  DAILY kubectl:                                                │
│  get all -n ns     describe pod    logs -f    exec -it bash  │
│  apply -f          rollout status  rollout undo  scale       │
│  set image         top pods        port-forward  diff        │
│                                                                │
│  ROLLING UPDATE:                                               │
│  maxSurge=1, maxUnavailable=0 = zero downtime                │
│  rollout undo = instant rollback                             │
│                                                                │
│  AWS MANAGED K8s:                                              │
│  EKS (Elastic Kubernetes Service)                             │
│  eksctl create cluster --name myapp --region us-east-1       │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the difference between a Pod and a Deployment?
2.  What is a ReplicaSet and how does it relate to a Deployment?
3.  What is the difference between livenessProbe
    and readinessProbe?
4.  What is a Service? What are the four types?
5.  What is an Ingress and when do you need it?
6.  What is the difference between ConfigMap and Secret?
7.  What is HPA and what metrics can it scale on?
8.  What is the difference between Deployment and StatefulSet?
9.  What does maxSurge: 1, maxUnavailable: 0 mean?
10. How do you rollback a failed deployment?
11. What is a Namespace and why use them?
12. What is Helm and why use it?
13. What is a PersistentVolumeClaim?
14. What is resource requests vs limits?
```

---

## 🛠️ Practice Exercises

Bash

```
# Exercise 1: Deploy on minikube
# Deploy your FastAPI app on local Kubernetes:
# a) Write Deployment, Service, ConfigMap, Secret YAML
# b) kubectl apply -f k8s/
# c) Access via minikube service myapp-service --url
# d) Scale to 5 replicas: kubectl scale deployment myapp --replicas=5
# e) Update image: kubectl set image deployment/myapp myapp=myapp:v2
# f) Watch rolling update: kubectl rollout status deployment/myapp
# g) Rollback: kubectl rollout undo deployment/myapp

# Exercise 2: HPA in Action
# a) Install metrics-server in minikube
# b) Create HPA targeting 50% CPU
# c) Generate load with: kubectl run load --image=busybox
#    -it --rm -- /bin/sh -c "while true; do wget -q -O- http://myapp-service; done"
# d) Watch HPA scale: kubectl get hpa --watch
# e) Stop load, watch scale-down

# Exercise 3: Health Checks
# a) Add liveness and readiness probes to your deployment
# b) Deploy a version that fails health checks
# c) Watch K8s NOT route traffic to it (readiness)
# d) Watch K8s restart it (liveness)
# e) Deploy a fixed version, verify gradual traffic shift

# Exercise 4: Complete Stack
# Deploy full stack on minikube:
# - FastAPI app (Deployment + Service)
# - PostgreSQL (StatefulSet + PVC + Service)
# - Redis (StatefulSet + PVC + Service)
# - Nginx Ingress
# - ConfigMap for app config
# - Secrets for DB/Redis passwords
# - HPA for the app
# Test: full registration → login → create post flow

# Exercise 5: EKS Deployment
# (Requires AWS account + ~$0.10/hour)
# a) Install eksctl
# b) eksctl create cluster --name myapp --region us-east-1
# c) Apply your K8s manifests
# d) Install ALB Ingress Controller
# e) Install cert-manager
# f) Access via your domain with HTTPS
# g) eksctl delete cluster --name myapp (to stop billing)
```

---

## ✅ Phase 7.5 Complete!

**You now know:**

text

```
✅ Kubernetes architecture (control plane + worker nodes)
✅ Pods — containers, resource limits, health checks
✅ Deployments — replicas, rolling updates, rollbacks
✅ Services — ClusterIP, NodePort, LoadBalancer
✅ Ingress — HTTP routing, TLS with cert-manager
✅ ConfigMaps and Secrets — configuration management
✅ HPA — auto-scaling by CPU/RAM/custom metrics
✅ PersistentVolumeClaims — persistent storage
✅ StatefulSets — for databases
✅ Namespaces — isolation and resource quotas
✅ kubectl — daily operational commands
✅ Helm — package management
✅ CI/CD with Kubernetes (GitHub Actions)
✅ Local development with minikube
```
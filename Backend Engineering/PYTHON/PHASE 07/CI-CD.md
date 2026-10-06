Continuous Integration/Continuous Deployment
```
Without CI/CD (manual process):
  1. Developer writes code
  2. "I think it works on my machine"
  3. Manually SSH to server
  4. git pull
  5. pip install -r requirements.txt
  6. sudo systemctl restart myapp
  7. Pray it works in production
  8. It doesn't. 2am emergency.

With CI/CD (automated pipeline):
  1. Developer pushes code to GitHub
  2. Pipeline automatically:
     → Runs linting
     → Runs all tests
     → Checks coverage
     → Builds Docker image
     → Pushes to registry
     → Deploys to staging
     → Runs smoke tests
     → Deploys to production
  3. If ANY step fails → pipeline stops, team notified
  4. No broken code reaches production

Benefits:
  ✅ Catch bugs before production
  ✅ Consistent deployment process
  ✅ No "works on my machine"
  ✅ Fast feedback (minutes, not hours)
  ✅ Confidence to deploy frequently
  ✅ Full audit trail of every deployment
```

---

## Setup

Bash

```
# We'll use GitHub Actions (free, built into GitHub)
# Everything is in .github/workflows/ YAML files
# No installation needed — just push to GitHub

mkdir -p ~/projects/cicd_demo/.github/workflows
cd ~/projects/cicd_demo

# We need a real GitHub repository
git init
git remote add origin https://github.com/YOUR_USERNAME/cicd_demo.git
```

---

## 1. 🔄 CI/CD Concepts

text

```
PIPELINE STAGES (in order):

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  SOURCE → LINT → TEST → BUILD → PUSH → DEPLOY → VERIFY    │
│                                                             │
│  SOURCE:   Code pushed to GitHub                           │
│  LINT:     ruff, black, mypy (seconds)                     │
│  TEST:     pytest unit + integration (minutes)             │
│  BUILD:    docker build (minutes)                          │
│  PUSH:     docker push to registry (seconds)               │
│  DEPLOY:   update running containers (seconds)             │
│  VERIFY:   smoke tests on live system (seconds)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘

ENVIRONMENTS:
  Development:  your laptop
  Staging:      production-like, for final testing
  Production:   real users, real data

DEPLOYMENT STRATEGIES:
  Rolling:      update instances one by one (zero downtime)
  Blue/Green:   run two environments, switch traffic instantly
  Canary:       route 5% of traffic to new version, monitor, expand
```

---

## 2. 🏗️ GitHub Actions Architecture

YAML

```
# .github/workflows/example.yml
# Every workflow file has this structure:

name: Workflow Name           # display name in GitHub UI

on:                           # WHEN to trigger
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:                   # cron syntax
    - cron: '0 2 * * *'      # 2am daily

env:                          # global environment variables
  PYTHON_VERSION: "3.12"
  REGISTRY: ghcr.io

jobs:                         # WHAT to do (can run in parallel)
  job-name:
    name: Human Readable Name
    runs-on: ubuntu-latest    # which machine to use

    strategy:                 # matrix = multiple combinations
      matrix:
        python: ["3.11", "3.12"]

    steps:                    # sequential steps in the job
      - name: Step name
        uses: actions/checkout@v4     # use a pre-built action

      - name: Another step
        run: echo "raw shell command"  # run shell command

      - name: Multi-line command
        run: |
          echo "line 1"
          echo "line 2"
          python -m pytest tests/

      - name: Step with env
        env:
          MY_SECRET: ${{ secrets.MY_SECRET }}
        run: echo $MY_SECRET

      - name: Conditional step
        if: github.ref == 'refs/heads/main'
        run: echo "Only on main branch"
```

---

## 3. 🔍 Complete CI Pipeline

YAML

```
# .github/workflows/ci.yml
# Runs on every push and PR
# Purpose: catch bugs early, fast feedback

name: CI

on:
  push:
    branches: [main, develop, "feature/**", "fix/**"]
  pull_request:
    branches: [main, develop]

# Cancel duplicate runs for the same branch/PR
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

env:
  PYTHON_VERSION: "3.12"
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ═══════════════════════════════════════
  # JOB 1: Code Quality (fastest — run first)
  # ═══════════════════════════════════════
  quality:
    name: "1️⃣ Code Quality"
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python ${{ env.PYTHON_VERSION }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip                    # cache pip downloads

      - name: Install quality tools
        run: |
          python -m pip install --upgrade pip
          pip install ruff black mypy

      - name: Ruff (linting)
        run: ruff check src/ tests/ --output-format=github
        # --output-format=github: annotates PRs with errors inline!

      - name: Ruff (formatting check)
        run: ruff format --check src/ tests/

      - name: Black (format check)
        run: black --check --diff src/ tests/

      - name: mypy (type checking)
        run: |
          pip install -r requirements.txt
          mypy src/ --ignore-missing-imports --no-error-summary
        continue-on-error: true       # don't fail CI on type errors yet

      - name: Check for debug statements
        run: |
          ! grep -rn "breakpoint()\|pdb.set_trace()\|print(" src/

  # ═══════════════════════════════════════
  # JOB 2: Security Scanning
  # ═══════════════════════════════════════
  security:
    name: "2️⃣ Security Scan"
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip

      - name: Install security tools
        run: pip install bandit pip-audit safety

      - name: Bandit (code security)
        run: |
          bandit -r src/ \
            -f json \
            -o bandit-results.json \
            -ll \
            || true           # don't fail on warnings

      - name: pip-audit (dependency vulnerabilities)
        run: pip-audit -r requirements.txt --format json > pip-audit.json || true

      - name: Upload security reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: security-reports
          path: |
            bandit-results.json
            pip-audit.json
          retention-days: 30

      # Fail on high severity issues
      - name: Check bandit for high severity
        run: |
          python -c "
          import json, sys
          with open('bandit-results.json') as f:
              data = json.load(f)
          high = [r for r in data.get('results', [])
                  if r['issue_severity'] == 'HIGH']
          if high:
              print(f'Found {len(high)} HIGH severity issues!')
              sys.exit(1)
          "

  # ═══════════════════════════════════════
  # JOB 3: Unit Tests
  # ═══════════════════════════════════════
  unit-tests:
    name: "3️⃣ Unit Tests"
    runs-on: ubuntu-latest
    needs: quality            # only run if quality passes

    strategy:
      matrix:
        python-version: ["3.11", "3.12"]
      fail-fast: false        # run both even if one fails

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Install dependencies
        run: |
          pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-mock faker factory-boy

      - name: Run unit tests
        run: |
          pytest tests/unit/ \
            -v \
            --tb=short \
            --cov=src \
            --cov-report=xml:coverage-unit.xml \
            --cov-report=term-missing \
            --cov-fail-under=80 \
            -m "not integration" \
            --junitxml=junit-unit.xml

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        if: matrix.python-version == '3.12'
        with:
          file: coverage-unit.xml
          flags: unit
          name: unit-py${{ matrix.python-version }}
          fail_ci_if_error: false
        env:
          CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

      - name: Test results summary
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Unit Tests (${{ matrix.python-version }})
          path: junit-unit.xml
          reporter: java-junit

  # ═══════════════════════════════════════
  # JOB 4: Integration Tests
  # ═══════════════════════════════════════
  integration-tests:
    name: "4️⃣ Integration Tests"
    runs-on: ubuntu-latest
    needs: quality

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 10
          --health-start-period 30s

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    env:
      DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
      REDIS_URL: redis://localhost:6379/0
      SECRET_KEY: test-secret-key-for-ci-at-least-32-chars!!
      JWT_SECRET_KEY: test-jwt-secret-for-ci-at-least-32-chars!!
      APP_ENV: testing

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio httpx

      - name: Run database migrations
        run: alembic upgrade head

      - name: Seed test data (if needed)
        run: |
          if [ -f "scripts/seed_test_data.py" ]; then
            python scripts/seed_test_data.py
          fi

      - name: Run integration tests
        run: |
          pytest tests/integration/ \
            -v \
            --tb=short \
            --cov=src \
            --cov-report=xml:coverage-integration.xml \
            --junitxml=junit-integration.xml \
            -m "integration"

      - name: Upload artifacts
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: integration-results
          path: |
            coverage-integration.xml
            junit-integration.xml

  # ═══════════════════════════════════════
  # JOB 5: Build Docker Image
  # ═══════════════════════════════════════
  build:
    name: "5️⃣ Build Docker Image"
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests]

    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
      image-digest: ${{ steps.build.outputs.digest }}

    steps:
      - uses: actions/checkout@v4

      # Set up QEMU for multi-platform builds
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # Set up Docker Buildx (advanced build features)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Login to GitHub Container Registry
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Generate image tags and labels
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            # Tag with branch name
            type=ref,event=branch
            # Tag with PR number
            type=ref,event=pr
            # Tag with git sha (short)
            type=sha,prefix=sha-,format=short
            # Tag as latest when pushing to main
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}
            # Semantic version tags (if you push git tags)
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      # Build and push with layer caching
      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64   # multi-arch!
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          # Cache from GitHub Actions cache
          cache-from: type=gha
          cache-to: type=gha,mode=max
          # Build args
          build-args: |
            BUILD_DATE=${{ github.event.repository.updated_at }}
            VCS_REF=${{ github.sha }}
            VERSION=${{ steps.meta.outputs.version }}

      # Scan image for vulnerabilities
      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}
          format: sarif
          output: trivy-results.sarif
          severity: CRITICAL,HIGH
          exit-code: 0          # don't fail (0), just report

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: trivy-results.sarif

      - name: Image info summary
        run: |
          echo "### Docker Image Built 🐳" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Property | Value |" >> $GITHUB_STEP_SUMMARY
          echo "| --- | --- |" >> $GITHUB_STEP_SUMMARY
          echo "| Image | \`${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| Tags | \`${{ steps.meta.outputs.version }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| Digest | \`${{ steps.build.outputs.digest }}\` |" >> $GITHUB_STEP_SUMMARY
```

---

## 4. 🚀 CD Pipeline — Deploy to Staging

YAML

```
# .github/workflows/deploy-staging.yml
# Deploys automatically when CI passes on main branch

name: Deploy to Staging

on:
  workflow_run:
    workflows: ["CI"]         # run after CI workflow
    types: [completed]
    branches: [main]

  # Or trigger on push to main directly:
  # push:
  #   branches: [main]

jobs:
  # ═══════════════════════════════════════
  # Only deploy if CI passed
  # ═══════════════════════════════════════
  check-ci:
    name: Check CI Status
    runs-on: ubuntu-latest
    if: >
      github.event.workflow_run.conclusion == 'success' ||
      github.event_name == 'push'

    outputs:
      should-deploy: ${{ steps.check.outputs.should-deploy }}

    steps:
      - id: check
        run: echo "should-deploy=true" >> $GITHUB_OUTPUT

  # ═══════════════════════════════════════
  # Deploy to Staging
  # ═══════════════════════════════════════
  deploy-staging:
    name: "🚀 Deploy to Staging"
    runs-on: ubuntu-latest
    needs: check-ci
    if: needs.check-ci.outputs.should-deploy == 'true'

    environment:
      name: staging
      url: https://staging.myapp.com

    env:
      DEPLOY_HOST: ${{ secrets.STAGING_HOST }}
      DEPLOY_USER: ${{ secrets.STAGING_USER }}
      IMAGE_TAG: ${{ github.sha }}

    steps:
      - uses: actions/checkout@v4

      # ─── Method 1: SSH + Docker Compose ───────────────────
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          port: ${{ secrets.STAGING_PORT || 22 }}
          envs: IMAGE_TAG
          script: |
            set -e

            echo "=== Deployment starting ==="
            echo "Image tag: $IMAGE_TAG"

            # Navigate to app directory
            cd /opt/myapp

            # Login to registry
            echo "${{ secrets.GITHUB_TOKEN }}" | \
              docker login ghcr.io \
              -u ${{ github.actor }} \
              --password-stdin

            # Update environment variable for new image
            export IMAGE_TAG=$IMAGE_TAG

            # Pull new image
            docker compose pull app worker

            # Run migrations before switching traffic
            docker compose run --rm app alembic upgrade head

            # Deploy with zero downtime
            docker compose up -d \
              --no-deps \
              --remove-orphans \
              app worker

            # Wait for health check
            echo "Waiting for app to become healthy..."
            for i in $(seq 1 30); do
              if docker compose exec -T app \
                curl -sf http://localhost:8000/health > /dev/null; then
                echo "App is healthy!"
                break
              fi
              echo "Attempt $i/30..."
              sleep 5
            done

            # Verify deployment
            docker compose ps
            echo "=== Deployment complete ==="

      # ─── Verify deployment ────────────────────────────────
      - name: Smoke tests on staging
        run: |
          echo "Running smoke tests..."

          # Wait for DNS propagation / load balancer
          sleep 10

          # Test health endpoint
          HTTP_STATUS=$(curl -sf -o /dev/null -w "%{http_code}" \
            https://staging.myapp.com/health)
          echo "Health check: HTTP $HTTP_STATUS"
          test "$HTTP_STATUS" = "200"

          # Test API responds
          HTTP_STATUS=$(curl -sf -o /dev/null -w "%{http_code}" \
            https://staging.myapp.com/api/v1/posts)
          echo "API check: HTTP $HTTP_STATUS"
          test "$HTTP_STATUS" = "200"

          echo "✅ All smoke tests passed!"

      # ─── Notify on success ────────────────────────────────
      - name: Notify Slack on success
        if: success()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "text": "✅ Staging deployment successful!",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "✅ *Staging Deploy Successful*\n*Repo:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}\n*Commit:* `${{ github.sha }}`\n*Deployed by:* ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK

      # ─── Notify on failure ────────────────────────────────
      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "text": "❌ Staging deployment failed!",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "❌ *Staging Deploy FAILED*\n*Repo:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}\n*Commit:* `${{ github.sha }}`\n*Actor:* ${{ github.actor }}\n<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View logs>"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

---

## 5. 🏭 Production Deployment

YAML

```
# .github/workflows/deploy-production.yml
# Manual approval required before production deploy

name: Deploy to Production

on:
  # Manual trigger with inputs
  workflow_dispatch:
    inputs:
      image-tag:
        description: "Image tag to deploy (e.g., sha-abc1234)"
        required: true
        type: string
      reason:
        description: "Reason for deployment"
        required: true
        type: string
      skip-smoke-tests:
        description: "Skip smoke tests (emergency only)"
        required: false
        type: boolean
        default: false

  # Or auto-deploy on release
  # release:
  #   types: [published]

jobs:
  # ═══════════════════════════════════════
  # Validation gate
  # ═══════════════════════════════════════
  validate:
    name: "🔍 Validate Before Deploy"
    runs-on: ubuntu-latest

    steps:
      - name: Check deployer permissions
        run: |
          ALLOWED_USERS="alice bob carol"
          if echo "$ALLOWED_USERS" | grep -qw "${{ github.actor }}"; then
            echo "✅ ${{ github.actor }} is authorized to deploy"
          else
            echo "❌ ${{ github.actor }} is NOT authorized to deploy"
            exit 1
          fi

      - name: Log deployment intent
        run: |
          echo "### Production Deployment Intent 🚀" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Field | Value |" >> $GITHUB_STEP_SUMMARY
          echo "| --- | --- |" >> $GITHUB_STEP_SUMMARY
          echo "| Tag | \`${{ inputs.image-tag }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| Reason | ${{ inputs.reason }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Initiated by | @${{ github.actor }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Time | $(date -u) |" >> $GITHUB_STEP_SUMMARY

  # ═══════════════════════════════════════
  # Production Deploy (requires approval)
  # ═══════════════════════════════════════
  deploy-production:
    name: "🏭 Deploy to Production"
    runs-on: ubuntu-latest
    needs: validate

    environment:
      name: production            # GitHub environment with approval!
      url: https://myapp.com

    env:
      IMAGE_TAG: ${{ inputs.image-tag || github.sha }}

    steps:
      - uses: actions/checkout@v4

      # ─── Create deployment record ─────────────────────────
      - name: Create GitHub deployment
        uses: chrnorm/deployment-action@v2
        id: deployment
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production
          ref: ${{ env.IMAGE_TAG }}

      # ─── Deploy (Blue/Green strategy) ─────────────────────
      - name: Deploy with Blue/Green strategy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          envs: IMAGE_TAG
          script: |
            set -e
            cd /opt/myapp

            echo "=== Starting Blue/Green Deploy ==="
            echo "Image: $IMAGE_TAG"

            # Login to registry
            echo "${{ secrets.GITHUB_TOKEN }}" | \
              docker login ghcr.io \
              -u ${{ github.actor }} \
              --password-stdin

            # 1. Start "green" environment on different port
            export IMAGE_TAG=$IMAGE_TAG
            docker compose -f docker-compose.yml \
                           -f docker-compose.prod.yml \
              up -d \
              --scale app=2 \
              --no-recreate

            # 2. Run migrations
            docker compose run --rm app alembic upgrade head

            # 3. Health check green instances
            sleep 30
            docker compose exec -T app \
              curl -sf http://localhost:8000/health

            # 4. Switch Nginx to new containers
            docker compose exec nginx nginx -s reload

            # 5. Remove old containers
            docker compose up -d --no-deps app

            echo "=== Blue/Green Deploy Complete ==="

      # ─── Production smoke tests ───────────────────────────
      - name: Production smoke tests
        if: ${{ !inputs.skip-smoke-tests }}
        run: |
          echo "Running production smoke tests..."

          # Critical endpoint checks
          for endpoint in /health /api/v1/posts; do
            STATUS=$(curl -sf -o /dev/null -w "%{http_code}" \
              "https://myapp.com$endpoint")
            echo "$endpoint: HTTP $STATUS"
            if [ "$STATUS" != "200" ]; then
              echo "❌ FAILED: $endpoint returned $STATUS"
              exit 1
            fi
          done

          # Performance check
          RESPONSE_TIME=$(curl -sf -o /dev/null \
            -w "%{time_total}" \
            https://myapp.com/health)
          echo "Response time: ${RESPONSE_TIME}s"

          # Fail if > 2 seconds
          python3 -c "
          t = float('$RESPONSE_TIME')
          if t > 2.0:
              print(f'❌ Response too slow: {t}s')
              exit(1)
          print(f'✅ Response time OK: {t}s')
          "

          echo "✅ All production smoke tests passed!"

      # ─── Update deployment status ─────────────────────────
      - name: Update deployment status (success)
        if: success()
        uses: chrnorm/deployment-status@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          deployment-id: ${{ steps.deployment.outputs.deployment_id }}
          state: success
          environment-url: https://myapp.com

      - name: Update deployment status (failure)
        if: failure()
        uses: chrnorm/deployment-status@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          deployment-id: ${{ steps.deployment.outputs.deployment_id }}
          state: failure

      # ─── Auto-rollback on failure ─────────────────────────
      - name: Rollback on failure
        if: failure()
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /opt/myapp
            echo "⚠️  ROLLING BACK to previous version..."

            # Bring back the previous image
            docker compose up -d \
              --no-deps \
              --no-build \
              app

            # Verify rollback
            sleep 10
            curl -sf https://myapp.com/health && \
              echo "✅ Rollback successful" || \
              echo "❌ Rollback also failed! Manual intervention needed!"

      # ─── Final notifications ──────────────────────────────
      - name: Notify production deployment
        if: always()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "text": "${{ job.status == 'success' && '✅' || '❌' }} Production deployment ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "${{ job.status == 'success' && '✅' || '❌' }} *Production Deploy: ${{ job.status }}*\n*Tag:* `${{ env.IMAGE_TAG }}`\n*By:* @${{ github.actor }}\n*Reason:* ${{ inputs.reason }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

---

## 6. 🔐 Secrets Management

YAML

```
# How secrets work in GitHub Actions:

# In GitHub UI:
# Settings → Secrets and variables → Actions → New repository secret
#
# Secret name: STAGING_SSH_KEY
# Secret value: -----BEGIN OPENSSH PRIVATE KEY----- ... (full private key)
#
# Access in workflow:
# ${{ secrets.STAGING_SSH_KEY }}
#
# Secrets are NEVER exposed in logs.
# GitHub masks them with ***

# Common secrets to set up:
# GITHUB_TOKEN         — auto-provided, for registry/deployment
# STAGING_HOST         — staging server IP/hostname
# STAGING_USER         — SSH username for staging
# STAGING_SSH_KEY      — private SSH key for staging
# STAGING_PORT         — SSH port (if not 22)
# PROD_HOST            — production server IP/hostname
# PROD_USER            — SSH username for production
# PROD_SSH_KEY         — private SSH key for production
# SLACK_WEBHOOK_URL    — for notifications
# CODECOV_TOKEN        — for coverage reporting
# SENTRY_DSN           — for error tracking

# Repository variables (non-secret config):
# Settings → Secrets and variables → Variables
# ACCESS in workflow: ${{ vars.STAGING_URL }}

# Environment-specific secrets (staging vs production):
# Settings → Environments → staging → Add secret
# These are only available when deploying to that environment

# Generating SSH key for CI:
ssh-keygen -t ed25519 -C "github-actions" -f ./ci_key -N ""
# ci_key     → add to STAGING_SSH_KEY secret (private)
# ci_key.pub → add to server's ~/.ssh/authorized_keys
```

---

## 7. 🎯 Workflow Best Practices

YAML

```
# .github/workflows/best_practices.yml

name: Best Practices Demo

on:
  push:
    branches: [main]

jobs:
  example:
    runs-on: ubuntu-latest
    timeout-minutes: 30          # ← prevent hung jobs
    
    steps:
      # ─── Checkout ─────────────────────────────────────────
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0           # full history for git commands
          # fetch-depth: 1 (default, shallow - faster)

      # ─── Dependency caching ───────────────────────────────
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip               # built-in pip caching

      # Manual cache (more control)
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          # Cache key includes hash of requirements
          key: pip-${{ hashFiles('requirements*.txt') }}
          restore-keys: |
            pip-

      # ─── Conditional steps ────────────────────────────────
      - name: Only on main
        if: github.ref == 'refs/heads/main'
        run: echo "This is main!"

      - name: Only on PRs
        if: github.event_name == 'pull_request'
        run: echo "This is a PR!"

      - name: Only if previous step succeeded
        if: success()
        run: echo "Previous step passed"

      - name: Always run (cleanup)
        if: always()
        run: echo "This always runs, even on failure"

      # ─── Step outputs ─────────────────────────────────────
      - name: Generate output
        id: version
        run: |
          VERSION=$(python -c "import src; print(src.__version__)")
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Use output
        run: echo "Version is ${{ steps.version.outputs.version }}"

      # ─── Environment files ────────────────────────────────
      - name: Add to PATH
        run: echo "/custom/path" >> $GITHUB_PATH

      - name: Set env var for all steps
        run: echo "MY_VAR=hello" >> $GITHUB_ENV

      - name: Use it
        run: echo $MY_VAR

      # ─── Job summary (shows in GitHub UI) ────────────────
      - name: Summary
        run: |
          echo "## Test Results 🧪" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Test | Status |" >> $GITHUB_STEP_SUMMARY
          echo "| --- | --- |" >> $GITHUB_STEP_SUMMARY
          echo "| Unit | ✅ |" >> $GITHUB_STEP_SUMMARY
          echo "| Integration | ✅ |" >> $GITHUB_STEP_SUMMARY

      # ─── Artifacts ────────────────────────────────────────
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            coverage.xml
            junit.xml
          retention-days: 7        # keep for 7 days

      # ─── Download artifact in another job ─────────────────
      # - name: Download artifact
      #   uses: actions/download-artifact@v4
      #   with:
      #     name: test-results

      # ─── Reusable workflow ────────────────────────────────
      # (define once, call from multiple workflows)
      # In .github/workflows/reusable-test.yml:
      # on:
      #   workflow_call:
      #     inputs:
      #       environment:
      #         required: true
      #         type: string
      #
      # Call it:
      # uses: ./.github/workflows/reusable-test.yml
      # with:
      #   environment: staging
```

---

## 8. 🔄 Deployment Strategies

YAML

```
# .github/workflows/strategies.yml
# Different deployment strategies explained

name: Deployment Strategies

jobs:
  # ─── ROLLING DEPLOYMENT ──────────────────────────────────────
  # Update instances one by one
  # Zero downtime if you have multiple instances
  rolling-deploy:
    name: Rolling Deployment
    runs-on: ubuntu-latest
    steps:
      - name: Rolling update
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /opt/myapp

            # Docker Compose rolling update
            # Updates one container at a time
            docker compose up -d --no-deps \
              --scale app=2 \
              app

            # Wait for health
            sleep 30

            # Scale back to normal
            docker compose up -d --no-deps \
              --scale app=1 \
              app

  # ─── BLUE/GREEN DEPLOYMENT ───────────────────────────────────
  # Run two identical environments
  # Switch traffic between them
  blue-green-deploy:
    name: Blue/Green Deployment
    runs-on: ubuntu-latest
    steps:
      - name: Blue/Green switch
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            # Determine current color
            CURRENT=$(cat /opt/myapp/.current-color 2>/dev/null || echo "blue")
            if [ "$CURRENT" = "blue" ]; then
              NEW="green"
              NEW_PORT=8001
            else
              NEW="blue"
              NEW_PORT=8000
            fi

            echo "Deploying to $NEW (port $NEW_PORT)"

            # Start new environment
            export COLOR=$NEW
            export PORT=$NEW_PORT
            docker compose -f docker-compose.$NEW.yml up -d

            # Wait and verify
            sleep 30
            curl -sf http://localhost:$NEW_PORT/health

            # Switch nginx to new environment
            sed -i "s/8000/$NEW_PORT/g" /etc/nginx/conf.d/myapp.conf
            nginx -s reload

            # Save new current color
            echo $NEW > /opt/myapp/.current-color

            echo "✅ Switched to $NEW environment"

  # ─── CANARY DEPLOYMENT ───────────────────────────────────────
  # Route small % of traffic to new version
  # Monitor, then gradually increase
  canary-deploy:
    name: Canary Deployment
    runs-on: ubuntu-latest
    steps:
      - name: Start canary (10% traffic)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            # Nginx upstream with weights = canary routing
            cat > /etc/nginx/conf.d/upstream.conf << 'EOF'
            upstream myapp {
                server 127.0.0.1:8000 weight=9;  # stable (90%)
                server 127.0.0.1:8001 weight=1;  # canary (10%)
            }
            EOF

            nginx -s reload
            echo "✅ Canary at 10% traffic"

      - name: Monitor canary (5 minutes)
        run: |
          echo "Monitoring canary for 5 minutes..."
          sleep 300

          # Check error rate in monitoring system
          ERROR_RATE=$(curl -s "https://metrics.myapp.com/api/error-rate?version=canary")
          echo "Canary error rate: $ERROR_RATE%"

          if python3 -c "exit(0 if float('$ERROR_RATE') < 1.0 else 1)"; then
            echo "✅ Canary healthy, proceeding with full rollout"
          else
            echo "❌ Canary unhealthy, rolling back"
            exit 1
          fi

      - name: Full rollout (100% traffic)
        run: |
          # All traffic to new version
          ssh ${{ secrets.PROD_USER }}@${{ secrets.PROD_HOST }} '
            cat > /etc/nginx/conf.d/upstream.conf << EOF
            upstream myapp {
                server 127.0.0.1:8001;  # new version, 100%
            }
            EOF
            nginx -s reload
          '
```

---

## 9. 📊 Pipeline Status and Notifications

YAML

```
# .github/workflows/notifications.yml

name: Notifications

on:
  workflow_run:
    workflows: ["CI", "Deploy to Staging", "Deploy to Production"]
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest

    steps:
      - name: Notify on Slack
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "${{ github.event.workflow_run.conclusion == 'success' && '✅' || '❌' }} *${{ github.event.workflow_run.name }}*\n${{ github.event.workflow_run.conclusion }} on `${{ github.event.workflow_run.head_branch }}`\n<${{ github.event.workflow_run.html_url }}|View Run>"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK

      - name: Create GitHub issue on failure
        if: github.event.workflow_run.conclusion == 'failure'
        uses: actions/github-script@v7
        with:
          script: |
            const run = context.payload.workflow_run;
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `❌ ${run.name} failed on ${run.head_branch}`,
              body: `
                ## Pipeline Failure

                **Workflow:** ${run.name}
                **Branch:** ${run.head_branch}
                **Commit:** ${run.head_sha}
                **Actor:** ${run.actor.login}

                [View Run](${run.html_url})

                Please investigate and fix.
              `,
              labels: ['ci-failure', 'bug'],
              assignees: [run.actor.login],
            });
```

---

## 10. 🛠️ Complete Pipeline Config

YAML

```
# .github/workflows/complete-pipeline.yml
# This is the single workflow that does everything
# for a production backend

name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, "release/**"]
    tags: ["v*"]
  pull_request:
    branches: [main]

concurrency:
  group: pipeline-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

env:
  PYTHON_VERSION: "3.12"
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:

  # ─── Stage 1: Quality Gate (fastest) ─────────────────────────
  quality:
    name: "Quality Gate"
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip
      - run: pip install ruff black mypy
      - run: ruff check src/ tests/
      - run: ruff format --check src/ tests/
      - run: black --check src/ tests/
      - run: pip install -r requirements.txt && mypy src/ --ignore-missing-imports
        continue-on-error: true

  # ─── Stage 2: Tests (parallel) ───────────────────────────────
  test:
    name: "Tests"
    runs-on: ubuntu-latest
    needs: quality
    timeout-minutes: 20

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ["5432:5432"]
        options: --health-cmd pg_isready --health-interval 5s --health-retries 10

      redis:
        image: redis:7-alpine
        ports: ["6379:6379"]
        options: --health-cmd "redis-cli ping" --health-interval 5s --health-retries 5

    env:
      DATABASE_URL: postgresql://test:test@localhost:5432/testdb
      REDIS_URL: redis://localhost:6379/0
      SECRET_KEY: test-secret-key-exactly-32-chars!!
      JWT_SECRET_KEY: test-jwt-secret-exactly-32-chars!!

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip

      - name: Install
        run: pip install -r requirements.txt -r requirements-dev.txt

      - name: Migrate
        run: alembic upgrade head

      - name: Test
        run: |
          pytest tests/ \
            -v \
            --cov=src \
            --cov-report=xml \
            --cov-fail-under=80 \
            --junitxml=junit.xml \
            -x \
            --tb=short

      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            coverage.xml
            junit.xml

  # ─── Stage 3: Build & Push ───────────────────────────────────
  build:
    name: "Build Image"
    runs-on: ubuntu-latest
    needs: test
    timeout-minutes: 30

    outputs:
      tags: ${{ steps.meta.outputs.tags }}
      digest: ${{ steps.build.outputs.digest }}
      version: ${{ steps.meta.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-qemu-action@v3
      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=sha-,format=short
            type=semver,pattern={{version}}
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─── Stage 4: Deploy Staging ─────────────────────────────────
  deploy-staging:
    name: "Deploy Staging"
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 15

    environment:
      name: staging
      url: https://staging.myapp.com

    env:
      IMAGE_TAG: ${{ needs.build.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          envs: IMAGE_TAG
          script: |
            set -e
            cd /opt/myapp
            export IMAGE_TAG=$IMAGE_TAG
            echo "${{ secrets.GITHUB_TOKEN }}" | \
              docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker compose pull app
            docker compose run --rm app alembic upgrade head
            docker compose up -d --no-deps app
            sleep 10
            docker compose exec -T app curl -sf http://localhost:8000/health

      - name: Smoke tests
        run: |
          for i in 1 2 3 4 5; do
            HTTP=$(curl -sf -o /dev/null -w "%{http_code}" \
              https://staging.myapp.com/health || echo "000")
            [ "$HTTP" = "200" ] && { echo "✅ Healthy"; break; }
            echo "Attempt $i: HTTP $HTTP, waiting..."
            sleep 10
          done
          [ "$HTTP" = "200" ] || exit 1

  # ─── Stage 5: Deploy Production (manual approval) ────────────
  deploy-production:
    name: "Deploy Production"
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    timeout-minutes: 30

    environment:
      name: production
      url: https://myapp.com

    env:
      IMAGE_TAG: ${{ needs.build.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          envs: IMAGE_TAG
          script: |
            set -e
            cd /opt/myapp

            # Backup current state
            docker compose exec -T db pg_dump \
              -U $DB_USER $DB_NAME \
              > /opt/backups/pre-deploy-$(date +%Y%m%d%H%M%S).sql

            # Deploy
            export IMAGE_TAG=$IMAGE_TAG
            echo "${{ secrets.GITHUB_TOKEN }}" | \
              docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker compose pull app
            docker compose run --rm app alembic upgrade head
            docker compose up -d --no-deps app

            # Verify
            sleep 15
            curl -sf https://myapp.com/health

      - name: Tag release in Git
        if: success()
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.git.createRef({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: `refs/tags/deployed-${new Date().toISOString().slice(0,10)}`,
              sha: context.sha
            });
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                    CI/CD COMPLETE FLOW                         │
│                                                                │
│  PUSH TO GITHUB                                                │
│       ↓                                                        │
│  ┌─── QUALITY (10s) ────────────────────────────────────────┐ │
│  │  ruff → black → mypy → security scan                     │ │
│  └──────────────────────────────────────────────────────────┘ │
│       ↓ (only if quality passes)                               │
│  ┌─── TESTS (5-10min) ──────────────────────────────────────┐ │
│  │  Unit tests → Integration tests (with PostgreSQL+Redis)  │ │
│  │  Coverage check (fail if < 80%)                          │ │
│  └──────────────────────────────────────────────────────────┘ │
│       ↓ (only if tests pass)                                   │
│  ┌─── BUILD (5-15min) ──────────────────────────────────────┐ │
│  │  docker build → security scan → push to registry         │ │
│  │  Tags: sha-abc123, latest, v1.2.3                        │ │
│  └──────────────────────────────────────────────────────────┘ │
│       ↓ (only on main branch)                                  │
│  ┌─── DEPLOY STAGING (auto) ────────────────────────────────┐ │
│  │  Pull image → migrate → restart → smoke tests            │ │
│  └──────────────────────────────────────────────────────────┘ │
│       ↓ (requires manual approval in GitHub)                   │
│  ┌─── DEPLOY PRODUCTION (gated) ────────────────────────────┐ │
│  │  Backup DB → pull image → migrate → restart → verify     │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  KEY CONCEPTS:                                                 │
│  needs: → job dependencies                                    │
│  environment: staging/production → approval gates            │
│  concurrency: → cancel in-progress on new push               │
│  secrets: → SSH keys, tokens, passwords                      │
│  services: → PostgreSQL, Redis in CI                         │
│  cache: → faster pip/docker builds                           │
│  matrix: → test multiple Python versions                     │
│  artifacts: → share files between jobs                       │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the difference between CI and CD?
2.  What is concurrency: cancel-in-progress good for?
3.  What does needs: do in a GitHub Actions job?
4.  How do you add PostgreSQL as a service in GitHub Actions?
5.  What is the difference between environment: (job level)
    and env: (step level)?
6.  How do you share data between jobs in a workflow?
7.  What is a GitHub environment and why is it useful?
8.  What does workflow_dispatch allow?
9.  What is the difference between rolling, blue/green,
    and canary deployment?
10. How do you reference secrets in a workflow?
11. What does continue-on-error: true do?
12. What is cache: in setup-python and why use it?
13. How do you trigger one workflow from another?
14. What does $GITHUB_STEP_SUMMARY do?
```

---

## 🛠️ Practice Exercises

YAML

```
# Exercise 1: Basic CI Pipeline
# Create a CI workflow for your FastAPI project that:
# - Triggers on push to main and all PRs
# - Has three jobs: lint, test, build
# - lint: ruff + black check (fail fast)
# - test: pytest with PostgreSQL service
# - build: docker build (don't push yet)
# - build only runs if tests pass
# Test it: make a PR with a linting error, verify CI fails

# Exercise 2: Docker Registry Push
# Extend Exercise 1:
# - Login to GitHub Container Registry
# - Tag image with: branch, sha, latest (on main)
# - Push to ghcr.io/your-username/your-repo
# - Add Trivy vulnerability scan
# - Make image public or handle private registry
# Verify: pull your image from the registry

# Exercise 3: Staging Deployment
# Set up a staging server (DigitalOcean $5 droplet or AWS EC2 free tier):
# 1. Create staging environment in GitHub
# 2. Add secrets: STAGING_HOST, STAGING_USER, STAGING_SSH_KEY
# 3. Write deploy-staging workflow:
#    - Triggers after CI passes on main
#    - SSH to server, pull image, run migrations, restart app
#    - Smoke test the staging URL
# 4. Test: push a commit, watch it deploy

# Exercise 4: Production with Approval
# Extend Exercise 3:
# 1. Create production environment with required reviewers
# 2. Write deploy-production workflow:
#    - workflow_dispatch trigger (manual)
#    - Input: image tag to deploy
#    - Requires production environment approval
#    - Deploy, verify, notify Slack
#    - Auto-rollback if smoke tests fail
# 3. Test the full flow: merge PR → auto staging → approve → production

# Exercise 5: Notifications and Observability
# Add to your pipeline:
# - Slack notification on staging deploy (success + failure)
# - Slack notification on production deploy
# - GitHub issue created on repeated failures
# - Test results posted as PR comment
# - Coverage report posted as PR comment
# - Pipeline duration tracking
# - Create a dashboard showing deployment history
```

---

## ✅ Phase 7.3 Complete!

**You now know:**

text

```
✅ CI/CD concepts and pipeline stages
✅ GitHub Actions architecture (workflows, jobs, steps)
✅ Quality gates (ruff, black, mypy, bandit)
✅ Unit and integration tests in CI with services
✅ Docker build and push to registry
✅ Multi-arch builds (amd64 + arm64)
✅ Image vulnerability scanning with Trivy
✅ Staging deployment via SSH + Docker Compose
✅ Production deployment with manual approval
✅ Blue/Green and Canary deployment strategies
✅ Secrets management in GitHub Actions
✅ Notifications (Slack, GitHub issues)
✅ Auto-rollback on deployment failure
✅ Caching for fast pipelines
✅ Matrix builds (multiple Python versions)
✅ Reusable workflows
✅ Complete CI/CD pipeline from push to production
```
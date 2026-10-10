# Interview

# Part 1 — Technical Tasks

## Q1. Create a directory named `release` and write version `1.0.0` into a `VERSION` file.

**Answer:**
```bash
mkdir -p ~/release
echo "1.0.0" > ~/release/VERSION
```

## Q2. Count the number of lines in `logs/access.log`.

**Answer:**
```bash
wc -l < logs/access.log
```

## Q3. Make `scripts/backup.sh` executable and run it.

**Answer:**
```bash
chmod 755 scripts/backup.sh
./scripts/backup.sh
```

## Q4. Write a Bash script that prints the application name, version and environment.

**Answer:**
```bash
#!/bin/bash
APP_NAME="growmar-web"
VERSION="v1.4.2"
ENVIRONMENT="${1:-staging}"

echo "Deploying $APP_NAME $VERSION to $ENVIRONMENT"
```

## Q5. Find the IP address with the highest number of requests.

**Answer:**
```bash
awk '{print $1}' logs/access.log | sort | uniq -c | sort -nr | head -1
```

## Q6. Count HTTP 5xx errors in an access log.

**Answer:**
```bash
awk '$9 >= 500 && $9 <= 599 {count++} END {print count+0}' logs/access.log
```

## Q7. Find files containing `DB_PASSWORD` under `projects/`.

**Answer:**
```bash
grep -rl "DB_PASSWORD" projects/
```

## Q8. Write a Python Dockerfile with dependency caching and a non-root user.

**Answer:**
```dockerfile
FROM python:3.12-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt \
    && useradd --create-home appuser
COPY . .
USER appuser
CMD ["python", "app.py"]
```

## Q9. Create a GitHub Actions workflow to install dependencies and run tests.

**Answer:**
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest
```

## Q10. Create a Kubernetes Deployment with three replicas and a readiness probe.

**Answer:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: growmar-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: growmar-web
  template:
    metadata:
      labels:
        app: growmar-web
    spec:
      containers:
        - name: web
          image: growmar/web:1.4.2
          ports:
            - containerPort: 8000
          readinessProbe:
            httpGet:
              path: /health
              port: 8000
```

## Q11. Write a Bash script to count `ERROR` entries in a log file.

**Answer:**
```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: summarize.sh <logfile>"
    exit 1
fi

if [ ! -f "$1" ]; then
    echo "File not found: $1"
    exit 2
fi

count=$(grep -c "ERROR" "$1")
echo "Errors found: $count"
```

## Q12. Create an S3 bucket using Terraform with versioning, encryption and public access blocked.

**Answer:**
```hcl
resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name
}

resource "aws_s3_bucket_versioning" "logs" {
  bucket = aws_s3_bucket.logs.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "logs" {
  bucket = aws_s3_bucket.logs.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Q13. Configure Docker Compose for a web application, PostgreSQL and Redis.

**Answer:**
```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:${POSTGRES_PASSWORD}@db:5432/postgres
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine

volumes:
  pgdata:
```

## Q14. Configure Nginx as a reverse proxy for two backend servers.

**Answer:**
```nginx
upstream growmar_app {
    server 10.0.1.10:8000;
    server 10.0.1.11:8000;
}

server {
    listen 80;
    server_name app.growmar.example;

    location / {
        proxy_pass http://growmar_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Q15. Write a deployment gate that succeeds only if every HTTP status is `200`.

**Answer:**
```bash
#!/bin/bash

total=$#
ok=0

for status in "$@"; do
    if [ "$status" = "200" ]; then
        ok=$((ok + 1))
    fi
done

echo "Healthy: $ok/$total"

if [ "$total" -gt 0 ] && [ "$ok" -eq "$total" ]; then
    echo "DEPLOY OK"
    exit 0
else
    echo "DEPLOY BLOCKED"
    exit 1
fi
```

## Q16. Create a Prometheus alert for a high HTTP 5xx error rate.

**Answer:**
```yaml
groups:
  - name: growmar-web
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High HTTP error rate detected"
```

## Q17. Nginx returns HTTP 502. How would you troubleshoot it?

**Answer:**
- Check whether the backend is running.
- Verify the backend port.
- Check connectivity between Nginx and the backend.
- Inspect application and Nginx logs.
- Test the backend directly.

## Q18. A Kubernetes pod is in `CrashLoopBackOff`. How do you inspect previous container logs?

**Answer:**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

## Q19. Find the most frequently requested URL that returns HTTP 404.

**Answer:**
```bash
awk '$9 == 404 {print $7}' logs/access.log | sort | uniq -c | sort -nr | head -1
```

## Q20. Find degraded production servers in a CSV file.

**Answer:**
```bash
awk -F',' '$3 == "prod" && $6 == "degraded" {print $1}' data/servers.csv | sort
```

## Q21. An API key is committed to a public repository. What should you do?

**Answer:**
1. Revoke or rotate the key immediately.
2. Check for unauthorized usage.
3. Remove the secret from the repository and clean history if necessary.
4. Use a secrets manager or encrypted CI/CD secrets.
5. Add local secret files to `.gitignore`.

## Q22. How can you prevent a production deployment when integration tests fail?

**Answer:** Make the integration tests a required CI/CD pipeline gate. Fix flaky tests and block deployment when required checks fail.

## Q23. Production latency increases after a new release. What should you do?

**Answer:** Check monitoring and logs, assess user impact, roll back if the release is responsible and rollback is safe, verify recovery, and investigate the root cause.

## Q24. How would you make a production deployment safer?

**Answer:** Use staging tests, gradual rollouts such as canary deployments, health checks, monitoring, and a tested rollback plan.

## Q25. How would you troubleshoot an unfamiliar deployment or infrastructure failure?

**Answer:** Identify the error, check logs and metrics, inspect recent changes, verify networking and permissions, isolate the failing component, restore service if necessary, and document the root cause.

---

# Part 2 —  40 MCQs

## Technical MCQs

### Q1. A server has free disk space but cannot create a new file. What should you check?

**Answer:** `df -i`

**Reason:** The filesystem may have exhausted its inodes.

### Q2. Which permission gives the owner full access, group read/execute, and others no access?

**Answer:** `chmod 750 filename`

### Q3. Which Linux signal cannot be caught or ignored?

**Answer:** `SIGKILL` (signal 9).

### Q4. How should you undo a faulty commit already pushed to shared `main`?

**Answer:** `git revert <commit>`

### Q5. How do you combine the last three local commits?

**Answer:** `git rebase -i HEAD~3`

### Q6. What is a good way to reduce Docker image size?

**Answer:** Use an appropriate minimal base image and multi-stage builds.

### Q7. A container exits immediately. Which command should you use?

**Answer:** `docker logs <container>`

### Q8. How can container data survive container removal?

**Answer:** Use a named Docker volume.

### Q9. How do you view logs from the previous failed Kubernetes container?

**Answer:** `kubectl logs <pod-name> --previous`

### Q10. Which Kubernetes object maintains desired replicas and supports rolling updates?

**Answer:** Deployment.

### Q11. Which Service type provides internal cluster access by default?

**Answer:** ClusterIP.

### Q12. Which Kubernetes object is designed to hold sensitive configuration values?

**Answer:** Secret.

### Q13. What happens when a readiness probe fails?

**Answer:** The pod stops receiving normal Service traffic until it becomes ready again.

### Q14. Which strategy releases a new version to a small percentage of users first?

**Answer:** Canary deployment.

### Q15. Where should CI/CD cloud credentials be stored?

**Answer:** In an encrypted secrets store or through a suitable identity mechanism.

### Q16. What is the difference between Continuous Delivery and Continuous Deployment?

**Answer:** Continuous Delivery prepares changes for release, often with manual approval. Continuous Deployment automatically releases changes that pass all required checks.

### Q17. Which Terraform file commonly contains resource definitions?

**Answer:** `main.tf`.

### Q18. What does `terraform plan` do?

**Answer:** Previews infrastructure changes without applying them.

### Q19. How should a team share Terraform state?

**Answer:** Use a remote backend with access controls and state locking.

### Q20. What does idempotency mean in Ansible?

**Answer:** Repeating an operation maintains the desired state without unnecessary changes.

### Q21. What is the principle of least privilege in IAM?

**Answer:** Grant only the permissions needed for the task.

### Q22. Which AWS service distributes traffic across EC2 instances in multiple Availability Zones?

**Answer:** Elastic Load Balancing (ELB).

### Q23. What is the purpose of an Auto Scaling Group?

**Answer:** Maintain the desired EC2 capacity and scale according to configured policies.

### Q24. Which tools are commonly used together for metrics and dashboards?

**Answer:** Prometheus and Grafana.

### Q25. What does HTTP 502 usually indicate?

**Answer:** An invalid upstream response or a communication problem with the backend.

## Workplace MCQs

### Q26. A deployment fails on Friday evening and the cause is unclear. What should you do?

**Answer:** Inform the team, then investigate.

### Q27. A teammate's configuration change causes an outage. What is the best response?

**Answer:** Alert the team, help restore service, and review the cause together later.

### Q28. Two managers assign conflicting urgent tasks. What should you do?

**Answer:** Ask both to agree on priorities and share your current workload.

### Q29. An API key is committed to a public repository. What should you do first?

**Answer:** Revoke or rotate the exposed key immediately.

### Q30. You need to use an unfamiliar tool under a tight deadline. What should you do?

**Answer:** Read the documentation, ask focused questions, and share progress early.

### Q31. How should you explain an outage to a non-technical client?

**Answer:** Explain the impact, say the fix is in progress, and provide an estimated resolution time in plain language.

### Q32. What is the best status-email structure?

**Answer:** Clear subject, key summary first, then actions and next steps.

### Q33. What should you do when you disagree with a senior colleague?

**Answer:** Respectfully explain your reasoning and alternative with evidence.

### Q34. Which behavior demonstrates active listening?

**Answer:** Paraphrasing to confirm your understanding.

### Q35. How should you respond to feedback that feels unfair?

**Answer:** Listen, ask for examples, and act on valid points.

## Numerical Aptitude

### Q36. Find the next number: 3, 6, 12, 24, __.

**Answer:** 48.

**Reason:** Each number is multiplied by 2.

### Q37. A train travels at 60 km/h. How long does it take to travel 150 km?

**Answer:** 2.5 hours (2 hours 30 minutes).

### Q38. Eight servers handle 800 requests per minute. How many servers are needed for 2,400 requests per minute at the same rate?

**Answer:** 24 servers.

### Q39. Calculate 20% of 250 plus 10% of 400.

**Answer:** 90.

**Calculation:** 50 + 40 = 90.

### Q40. Which one is different: Kubernetes, Nomad, Docker Swarm, Git?

**Answer:** Git.

**Reason:** The first three are container orchestration tools; Git is a version control system.

---

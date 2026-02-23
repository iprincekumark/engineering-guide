# 17 — DevOps, CI/CD & Containerization

> **Goal:** Understand Docker, Kubernetes basics, CI/CD pipelines, and cloud deployment essentials for Java backend developers.

---

## Table of Contents

1. [Docker for Java Apps](#1-docker-for-java-apps)
2. [Docker Compose](#2-docker-compose)
3. [Kubernetes Basics](#3-kubernetes-basics)
4. [CI/CD Pipelines](#4-cicd-pipelines)
5. [Infrastructure Essentials](#5-infrastructure-essentials)
6. [Cloud Deployment Basics](#6-cloud-deployment-basics)
7. [Interview Notes](#7-interview-notes)
8. [Summary](#8-summary)
9. [References](#9-references)

---

## 1. Docker for Java Apps

**Definition:**
Docker packages your application and all its dependencies into a lightweight, portable container that runs consistently on any platform — your laptop, staging, or production.

**Why it exists:**
"Works on my machine" — the most common developer excuse. Docker eliminates this by bundling the exact JVM, libraries, config, and OS layers into a reproducible image.

**Real-life analogy:**
A shipping container. Regardless of what's inside (furniture, electronics, food), the container has a standard size, fits on any truck/ship, and protects the contents.

**Dockerfile for Spring Boot:**
```dockerfile
# Multi-stage build for smaller image
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY mvnw .
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline
COPY src src
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# Non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
ENTRYPOINT ["java", "-Xms512m", "-Xmx512m", "-jar", "app.jar"]
```

**Key practices:**
- Use multi-stage builds (build + runtime stages)
- Use JRE (not JDK) for runtime — smaller image
- Run as non-root user
- Use `.dockerignore` to exclude unnecessary files
- Set JVM memory flags explicitly
- Use specific image tags, never `latest`

---

## 2. Docker Compose

**Definition:**
Docker Compose defines multi-container applications in a single YAML file. Spin up your app + PostgreSQL + Redis + Kafka locally with one command.

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
      SPRING_DATA_REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

```bash
docker compose up -d     # Start all services
docker compose logs -f   # View logs
docker compose down      # Stop and remove
```

---

## 3. Kubernetes Basics

**Definition:**
Kubernetes (K8s) is a container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Key concepts for backend developers:**

| Concept | What It Does | Analogy |
|---------|-------------|---------|
| Pod | Smallest deployable unit (1+ containers) | A single apartment |
| Deployment | Manages pod replicas, rolling updates | Building management |
| Service | Stable network endpoint for pods | Building address |
| ConfigMap | External configuration | Instruction manual |
| Secret | Sensitive configuration | Safe/vault |
| Ingress | External HTTP routing | Building entrance |
| HPA | Auto-scaling based on metrics | Hiring more staff when busy |

**Example Deployment:**
```yaml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: myregistry/user-service:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 15
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
```

---

## 4. CI/CD Pipelines

**Definition:**
CI/CD (Continuous Integration / Continuous Delivery) automates the build → test → deploy pipeline. Every code push triggers automated testing and deployment.

**GitHub Actions example:**
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven

      - name: Build & Test
        run: mvn clean verify
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb

      - name: Build Docker Image
        if: github.ref == 'refs/heads/main'
        run: |
          docker build -t myregistry/user-service:${{ github.sha }} .
          docker push myregistry/user-service:${{ github.sha }}

  deploy:
    needs: build-and-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/user-service \
            user-service=myregistry/user-service:${{ github.sha }}
```

**Pipeline stages:**
```
Code Push → Build → Unit Tests → Integration Tests → Docker Build → Push Image → Deploy
```

---

## 5. Infrastructure Essentials

| Component | Purpose | Tool |
|-----------|---------|------|
| Reverse Proxy | TLS termination, routing | Nginx, Traefik |
| Load Balancer | Distribute traffic | AWS ALB, Nginx |
| DNS | Domain name resolution | Route 53, Cloudflare |
| SSL/TLS | Encryption in transit | Let's Encrypt, ACM |
| Secrets Management | Store credentials | Vault, AWS Secrets Manager |
| Database (managed) | Relational DB | AWS RDS, Cloud SQL |
| Object Storage | Files, images | S3, GCS |
| CDN | Static content delivery | CloudFront, Cloudflare |

---

## 6. Cloud Deployment Basics

**AWS essentials for Java backend:**

| Service | What For |
|---------|----------|
| EC2 | Virtual machines |
| ECS / EKS | Container orchestration |
| RDS | Managed PostgreSQL/MySQL |
| ElastiCache | Managed Redis |
| S3 | File storage |
| SQS / SNS | Messaging |
| CloudWatch | Monitoring & logs |
| ALB | Load balancing |

---

## 7. Interview Notes

1. **How do you dockerize a Spring Boot app?** — Multi-stage Dockerfile: build with JDK, run with JRE. Set JVM flags, non-root user.
2. **What is the difference between Docker and Kubernetes?** — Docker packages apps in containers. Kubernetes orchestrates containers (scaling, healing, networking).
3. **What does your CI/CD pipeline look like?** — Push → build → unit tests → integration tests → Docker build → push → deploy to K8s.
4. **What are readiness and liveness probes?** — Readiness: is the app ready for traffic? Liveness: is the app alive? Both use Actuator health endpoints.
5. **How do you manage secrets in production?** — Environment variables, Kubernetes Secrets, or a vault (HashiCorp Vault, AWS Secrets Manager). Never in code.

---

## 8. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Docker | Multi-stage builds, JRE runtime, non-root user |
| Docker Compose | Local development with app + DB + Redis |
| Kubernetes | Deployments, Services, probes, HPA |
| CI/CD | Automated build → test → deploy pipeline |
| Cloud | Managed services reduce operational burden |

---

## 9. References

- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Spring Boot Docker Guide](https://spring.io/guides/topicals/spring-boot-docker)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS Getting Started](https://aws.amazon.com/getting-started/)
- [The Twelve-Factor App](https://12factor.net/)

---

> **Previous:** [← 16 — Testing Strategy Deep Dive](./16_Testing_Strategy_Deep_Dive.md)
> **Next:** [00 — Verification Checklist →](./00_Verification_Checklist.md)

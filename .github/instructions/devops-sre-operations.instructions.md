---
description: "Use when DevOps/SRE prepares deployment readiness: Dockerfile optimization, CI/CD pipeline design, cloud infrastructure provisioning, Kubernetes deployment patterns, secret management, observability setup, and incident response runbooks."
name: "DevOps SRE Operations Standards"
---

# DevOps SRE Operations Standards Instruction

## Table of Contents
1. Dockerfile Standards
2. CI/CD Pipeline Design
3. Cloud Infrastructure Standards
4. Kubernetes Deployment Patterns
5. Secret Management Protocol
6. Observability and Monitoring Setup
7. Environment Promotion Flow
8. Incident Response Runbook Template
9. Release Management Protocol
10. Output Quality Gate

## 1. Dockerfile Standards

### Multi-Stage Build Template (Java Spring Boot)
```dockerfile
# Stage 1: Build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests -B

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -D appuser
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -q --spider http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Dockerfile Rules
| Rule | Rationale |
|---|---|
| Use multi-stage builds | Reduce image size, exclude build tools |
| Use specific base image tags | Prevent unexpected breaking changes |
| Run as non-root user | Security hardening |
| Include HEALTHCHECK | Container orchestrator can detect unhealthy instances |
| Use `.dockerignore` | Exclude unnecessary files from build context |
| Order layers by change frequency | Maximize cache hit rate |
| Pin dependency versions | Reproducible builds |

### Image Naming Convention
```
<registry>/<project>/<service>:<version>-<git-short-sha>
Example: ghcr.io/myorg/user-service:1.2.0-abc1234
```

## 2. CI/CD Pipeline Design

### Pipeline Stages (Mandatory Order)
```mermaid
graph LR
    A[Checkout] --> B[Build]
    B --> C[Unit Test]
    C --> D[Integration Test]
    D --> E[Security Scan]
    E --> F[Build Image]
    F --> G[Push Image]
    G --> H[Deploy Staging]
    H --> I[Smoke Test]
    I --> J[Manual Approval]
    J --> K[Deploy Production]
    K --> L[Health Verify]
```

### Stage Definitions
| Stage | Tool | Gate Criteria | Failure Action |
|---|---|---|---|
| Build | Maven/Gradle/npm | Compilation success | Block pipeline |
| Unit Test | JUnit/Jest | All tests pass, coverage ≥ 70% | Block pipeline |
| Integration Test | Testcontainers/REST Assured | All tests pass | Block pipeline |
| Security Scan | SAST + Dependency scan | No Critical findings | Block pipeline |
| Build Image | Docker | Image builds cleanly | Block pipeline |
| Push Image | Container Registry | Push succeeds | Block pipeline |
| Deploy Staging | K8s/Terraform | Deployment healthy | Block pipeline |
| Smoke Test | Automated suite | Critical paths pass | Block pipeline |
| Manual Approval | Human gate | Reviewer approves | Wait |
| Deploy Production | K8s/Terraform | Deployment healthy | Auto-rollback |
| Health Verify | Health checks | All checks pass for 5 min | Auto-rollback |

### GitHub Actions Convention
- Workflow files: `.github/workflows/ci.yml`, `.github/workflows/cd-staging.yml`, `.github/workflows/cd-production.yml`.
- Use reusable workflows for shared logic.
- Pin action versions with SHA (not `@latest`).
- Store secrets in GitHub Secrets, never in workflow files.

## 3. Cloud Infrastructure Standards

### Infrastructure as Code (IaC)
- Use **Terraform** as default IaC tool.
- Organize by environment and module.

```
infrastructure/
├── modules/
│   ├── networking/      # VPC, subnets, security groups
│   ├── compute/         # ECS/EKS, EC2, App Runner
│   ├── database/        # RDS, ElastiCache
│   ├── storage/         # S3, EFS
│   └── monitoring/      # CloudWatch, alerts
├── environments/
│   ├── dev/
│   │   └── main.tf
│   ├── staging/
│   │   └── main.tf
│   └── production/
│       └── main.tf
└── backend.tf           # Remote state config
```

### Terraform Rules
| Rule | Rationale |
|---|---|
| Remote state backend (S3 + DynamoDB) | State file safety and locking |
| State file per environment | Isolation between environments |
| Module reuse across environments | Consistency and DRY |
| `terraform plan` before `apply` | Catch unintended changes |
| Lock provider versions | Reproducible infrastructure |
| Tag all resources | Cost tracking and ownership |
| Output critical values | Downstream automation |

### Resource Tagging Convention
| Tag | Value | Purpose |
|---|---|---|
| `Project` | `<project-name>` | Ownership |
| `Environment` | `dev\|staging\|production` | Environment separation |
| `ManagedBy` | `terraform` | Automation tracking |
| `CostCenter` | `<team-code>` | Cost allocation |
| `CreatedAt` | `<iso-date>` | Lifecycle tracking |

## 4. Kubernetes Deployment Patterns

### Deployment Strategy Selection
| Strategy | When to Use | Risk Level |
|---|---|---|
| Rolling Update | Default for most services | Low |
| Blue/Green | Zero-downtime for critical services | Medium |
| Canary | High-risk changes, gradual rollout | Low-Medium |
| Recreate | Stateful services, breaking changes | High (downtime) |

### Kubernetes Manifest Standards
- Use `Deployment` for stateless services.
- Use `StatefulSet` for stateful services (DB, messaging).
- Always define:
  - `resources.requests` and `resources.limits` (CPU + Memory).
  - `livenessProbe` and `readinessProbe`.
  - `podDisruptionBudget` for production.
  - `topologySpreadConstraints` for availability.

### Resource Template
```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1000m"
    memory: "1Gi"

livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 60
  periodSeconds: 15

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### Auto-Scaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: <service>-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: <service>
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

## 5. Secret Management Protocol

### Hierarchy (Most to Least Preferred)
1. **Cloud Secret Manager** (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager)
2. **Kubernetes Secrets** (encrypted at rest with KMS)
3. **CI/CD Platform Secrets** (GitHub Secrets, GitLab CI Variables)
4. **Environment Variables** (via `.env` files NOT committed to git)

### Rules
- NEVER commit secrets to source code or git history.
- NEVER pass secrets as command-line arguments (visible in process list).
- ALWAYS rotate secrets periodically (minimum quarterly).
- ALWAYS use different secrets per environment.
- ALWAYS audit secret access logs.
- ALWAYS use `.gitignore` for `.env`, `*.pem`, `*.key` files.

### Secret Rotation Protocol
1. Generate new secret in secret manager.
2. Deploy application with support for both old and new secret.
3. Verify application works with new secret.
4. Revoke old secret.
5. Remove dual-secret support in next deployment.

## 6. Observability and Monitoring Setup

### Three Pillars
| Pillar | Tool Options | What to Capture |
|---|---|---|
| Logging | ELK Stack, CloudWatch Logs, Loki | Structured JSON logs with correlation IDs |
| Metrics | Prometheus + Grafana, CloudWatch Metrics | Golden signals, business KPIs |
| Tracing | Jaeger, Zipkin, X-Ray, OpenTelemetry | Request flow across services |

### Golden Signals (Mandatory)
| Signal | Metric | Alert Threshold |
|---|---|---|
| Latency | P95 response time | > 500ms sustained 5 min |
| Traffic | Requests per second | > 150% of baseline sustained 5 min |
| Errors | Error rate (5xx) | > 1% sustained 2 min |
| Saturation | CPU/Memory utilization | > 85% sustained 5 min |

### Health Check Endpoints
```
GET /actuator/health          # Overall health (UP/DOWN)
GET /actuator/health/liveness # K8s liveness (is the app alive?)
GET /actuator/health/readiness # K8s readiness (can it serve traffic?)
GET /actuator/info            # Build version and metadata
GET /actuator/prometheus      # Metrics endpoint for scraping
```

### Dashboard Requirements
- Service overview: uptime, error rate, latency, throughput.
- Per-endpoint breakdown: response time distribution, error codes.
- Infrastructure: CPU, memory, disk, network.
- Business metrics: active users, transactions per minute.

## 7. Environment Promotion Flow

```
Dev → Staging → Production
```

| Gate | Dev → Staging | Staging → Production |
|---|---|---|
| Build | ✅ Passes CI | ✅ Same artifact |
| Unit Tests | ✅ Pass | ✅ Same results |
| Integration Tests | ✅ Pass | ✅ Pass on staging |
| Security Scan | ✅ No Critical | ✅ No Critical/High |
| Smoke Tests | ⚪ Optional | ✅ Required on staging |
| Performance Tests | ⚪ Optional | ✅ Baseline met |
| Manual Approval | ⚪ Optional | ✅ Required |
| Rollback Plan | ✅ Documented | ✅ Tested |

### Environment Rules
- SAME Docker image promoted across environments (no rebuild).
- Configuration differences handled via environment variables only.
- Database migrations must be backward-compatible or pre-deployed.
- Staging MUST mirror production topology (scaled down).

## 8. Incident Response Runbook Template

```markdown
# Incident Runbook: [Service/Component Name]

## 1. Quick Reference
- Service: <name>
- Team: <owner>
- Escalation: <contacts>
- Dashboard: <link>
- Logs: <link>

## 2. Common Incidents

### INC-001: Service Unresponsive
- Symptoms: Health check failures, timeout errors.
- Check: Pod status, logs, resource usage.
- Action: Restart pods, check DB connections, verify dependencies.
- Escalate if: Restart does not resolve within 10 min.

### INC-002: High Error Rate
- Symptoms: >1% 5xx errors.
- Check: Application logs, dependency health, recent deployments.
- Action: Roll back last deployment if correlated, check DB locks.
- Escalate if: Error rate does not decrease within 15 min.

### INC-003: Database Connection Exhaustion
- Symptoms: Connection pool full, query timeouts.
- Check: Active connections, slow queries, connection leaks.
- Action: Kill long-running queries, restart connection pool, scale pool size.
- Escalate if: Connections continue to leak after pool restart.

## 3. Rollback Procedure
1. Identify last stable version.
2. Run: `kubectl rollout undo deployment/<service> -n <namespace>`
3. Verify health checks pass.
4. Notify team and document incident.

## 4. Post-Incident
- Write postmortem within 48 hours.
- Include: timeline, root cause, impact, remediation, preventive actions.
- Track follow-up actions in backlog.
```

## 9. Release Management Protocol

### Version Convention
Semantic Versioning: `MAJOR.MINOR.PATCH`
- MAJOR: Breaking API changes.
- MINOR: New features, backward-compatible.
- PATCH: Bug fixes, backward-compatible.

### Release Checklist
- [ ] All CI/CD pipeline stages passed.
- [ ] Security scan completed with no Critical findings.
- [ ] Performance baseline met on staging.
- [ ] Database migrations applied and verified on staging.
- [ ] Release notes prepared.
- [ ] Rollback plan documented and tested.
- [ ] Manual approval obtained.
- [ ] Monitoring dashboards and alerts active.
- [ ] On-call team notified of deployment window.

### Release Notes Template
```markdown
## Release v<X.Y.Z> — YYYY-MM-DD

### Added
- <new feature descriptions>

### Changed
- <modified behavior descriptions>

### Fixed
- <bug fix descriptions>

### Security
- <security improvements>

### Breaking Changes
- <breaking changes with migration guide>

### Known Issues
- <known issues with workarounds>
```

## 10. Output Quality Gate (Mandatory)
DevOps/SRE task is NOT complete unless:

- [ ] Dockerfiles follow multi-stage build and security standards.
- [ ] CI/CD pipeline covers build → test → scan → deploy → verify.
- [ ] Infrastructure defined as code with Terraform.
- [ ] Kubernetes manifests include resource limits, probes, and PDB.
- [ ] Secrets managed via secret manager (no hardcoded).
- [ ] Observability configured (logs, metrics, traces, alerts).
- [ ] Environment promotion flow documented.
- [ ] Incident runbook created or updated.
- [ ] Rollback plan documented and tested.
- [ ] `/PROJECT_STATE.md` updated with deployment status.

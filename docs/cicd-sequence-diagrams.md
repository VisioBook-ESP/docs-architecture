# Diagrammes de Séquence CI/CD - Projet Visiobook

Ce document contient les diagrammes de séquence Mermaid expliquant les différents flux CI/CD du projet Visiobook.

## 1. Flux CI/CD Principal - Vue d'ensemble

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as GitHub
    participant CI as GitHub Actions
    participant Reg as Docker Registry
    participant ArgoCD as ArgoCD
    participant K8s as Kubernetes
    participant Mon as Monitoring

    Dev->>Git: git push / pull request
    Git->>CI: Trigger workflow

    Note over CI: Stage: Validate
    CI->>CI: Lint code
    CI->>CI: Security scan
    CI->>CI: Helm validation

    Note over CI: Stage: Build & Test
    CI->>CI: npm/maven build
    CI->>CI: Unit tests
    CI->>CI: Integration tests

    Note over CI: Stage: Docker Build
    CI->>Reg: Build & push image
    Reg-->>CI: Image pushed successfully

    Note over CI: Stage: Deploy Staging
    CI->>ArgoCD: Update staging manifests
    ArgoCD->>K8s: Deploy to staging
    K8s-->>ArgoCD: Deployment status
    ArgoCD-->>CI: Staging deployed

    Note over CI: Stage: Integration Tests
    CI->>K8s: Run integration tests
    K8s-->>CI: Tests passed

    Note over CI: Stage: Deploy Production (Manual)
    Dev->>CI: Approve production deployment
    CI->>ArgoCD: Update production manifests
    ArgoCD->>K8s: Deploy to production
    K8s-->>ArgoCD: Deployment status
    ArgoCD-->>CI: Production deployed

    K8s->>Mon: Send metrics & logs
    Mon-->>Dev: Deployment notifications
```

## 2. Flux de Build et Tests Détaillé

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as GitHub
    participant CI as GitHub Actions
    participant SonarQ as SonarQube
    participant OWASP as OWASP ZAP
    participant Reg as Docker Registry
    participant TestDB as Test Database

    Dev->>Git: Push code changes
    Git->>CI: Trigger workflow

    Note over CI: Validation Stage
    CI->>CI: Checkout code
    CI->>CI: Install dependencies
    CI->>CI: ESLint / Prettier
    CI->>SonarQ: Code quality analysis
    SonarQ-->>CI: Quality gate result

    alt Quality gate failed
        CI-->>Dev: Pipeline failed - Quality issues
    else Quality gate passed
        Note over CI: Security Stage
        CI->>CI: npm audit / dependency check
        CI->>OWASP: Security vulnerability scan
        OWASP-->>CI: Security report

        alt Security issues found
            CI-->>Dev: Pipeline failed - Security issues
        else Security OK
            Note over CI: Build Stage
            CI->>CI: Build application
            CI->>CI: Generate artifacts

            Note over CI: Test Stage
            CI->>TestDB: Start test database
            CI->>CI: Run unit tests
            CI->>CI: Run integration tests
            CI->>CI: Generate coverage report
            CI->>TestDB: Stop test database

            alt Tests failed
                CI-->>Dev: Pipeline failed - Tests failed
            else Tests passed
                Note over CI: Docker Stage
                CI->>CI: Build Docker image
                CI->>CI: Security scan image
                CI->>Reg: Push image with tags
                Reg-->>CI: Image pushed successfully
                CI-->>Dev: Build successful
            end
        end
    end
```

## 3. Flux de Déploiement avec ArgoCD

```mermaid
sequenceDiagram
    participant CI as GitHub Actions
    participant GitOps as GitOps Repo
    participant ArgoCD as ArgoCD
    participant K8s as Kubernetes
    participant Helm as Helm
    participant Prom as Prometheus
    participant Alert as Alertmanager

    CI->>GitOps: Update Helm values (image tag)
    GitOps-->>ArgoCD: GitHub webhook notification

    Note over ArgoCD: Sync Detection
    ArgoCD->>GitOps: Fetch latest changes
    ArgoCD->>ArgoCD: Compare desired vs current state

    alt No changes detected
        ArgoCD->>ArgoCD: Skip sync
    else Changes detected
        Note over ArgoCD: Sync Process
        ArgoCD->>Helm: Generate manifests
        Helm-->>ArgoCD: Kubernetes manifests

        ArgoCD->>K8s: Apply manifests
        K8s->>K8s: Create/Update resources

        Note over K8s: Health Checks
        K8s->>K8s: Readiness probes
        K8s->>K8s: Liveness probes

        alt Health checks failed
            K8s-->>ArgoCD: Deployment failed
            ArgoCD->>K8s: Rollback to previous version
            ArgoCD->>Alert: Send failure alert
            Alert-->>CI: Deployment failed notification
        else Health checks passed
            K8s-->>ArgoCD: Deployment successful
            K8s->>Prom: Start metrics collection
            ArgoCD-->>CI: Deployment successful

            Note over ArgoCD: Post-deployment
            ArgoCD->>ArgoCD: Update sync status
            ArgoCD->>Prom: Send deployment metrics
        end
    end
```

## 4. Flux Spécifique Services IA avec GPU

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant CI as GitHub Actions
    participant GPURunner as GPU Runner
    participant NvidiaReg as NVIDIA Registry
    participant Reg as Docker Registry
    participant K8s as Kubernetes
    participant GPUNode as GPU Node
    participant GPUMon as GPU Monitoring

    Dev->>CI: Push AI service code
    CI->>CI: Detect AI service (GPU required)

    Note over CI: GPU Build Stage
    CI->>GPURunner: Allocate GPU runner
    GPURunner->>NvidiaReg: Pull CUDA base images
    GPURunner->>GPURunner: Build with NVIDIA drivers
    GPURunner->>GPURunner: Install ML frameworks (PyTorch/TF)

    Note over GPURunner: AI Model Tests
    GPURunner->>GPURunner: Load test models
    GPURunner->>GPURunner: Run inference tests
    GPURunner->>GPURunner: Performance benchmarks
    GPURunner->>GPURunner: Memory usage tests

    alt GPU tests failed
        GPURunner-->>CI: Tests failed - GPU issues
        CI-->>Dev: Pipeline failed
    else GPU tests passed
        GPURunner->>Reg: Push GPU-enabled image
        Reg-->>CI: Image pushed successfully

        Note over CI: GPU Deployment
        CI->>K8s: Deploy with GPU requirements
        K8s->>K8s: Schedule on GPU nodes
        K8s->>GPUNode: Allocate GPU resources

        Note over GPUNode: GPU Resource Allocation
        GPUNode->>GPUNode: Reserve GPU memory
        GPUNode->>GPUNode: Set GPU limits
        GPUNode->>GPUNode: Start container with GPU access

        Note over GPUNode: Health Checks
        GPUNode->>GPUNode: GPU availability check
        GPUNode->>GPUNode: Model loading check
        GPUNode->>GPUNode: Inference endpoint check

        alt GPU health checks failed
            GPUNode-->>K8s: GPU deployment failed
            K8s->>K8s: Reschedule on different GPU node
        else GPU health checks passed
            GPUNode-->>K8s: GPU service ready
            GPUNode->>GPUMon: Start GPU metrics collection

            Note over GPUMon: GPU Monitoring
            GPUMon->>GPUMon: GPU utilization
            GPUMon->>GPUMon: GPU memory usage
            GPUMon->>GPUMon: GPU temperature
            GPUMon->>GPUMon: Model inference latency

            K8s-->>CI: GPU deployment successful
            CI-->>Dev: AI service deployed with GPU
        end
    end
```

## 5. Flux de Monitoring et Alerting

```mermaid
sequenceDiagram
    participant K8s as Kubernetes
    participant Prom as Prometheus
    participant Grafana as Grafana
    participant Alert as Alertmanager
    participant Slack as Slack
    participant PagerDuty as PagerDuty
    participant DevTeam as Dev Team

    Note over K8s: Post-Deployment Monitoring
    K8s->>Prom: Scrape metrics (/metrics)
    K8s->>Prom: Send custom business metrics

    Note over Prom: Metrics Collection
    Prom->>Prom: Store time series data
    Prom->>Prom: Evaluate alerting rules

    Note over Prom: Alert Evaluation
    alt Normal conditions
        Prom->>Grafana: Update dashboards
        Grafana->>Grafana: Display green status
    else Alert conditions met
        Prom->>Alert: Fire alert

        Note over Alert: Alert Processing
        Alert->>Alert: Group similar alerts
        Alert->>Alert: Apply routing rules

        alt Critical alert (P1)
            Alert->>PagerDuty: Send critical alert
            PagerDuty->>DevTeam: Call/SMS on-call engineer
            Alert->>Slack: Send to #critical-alerts
        else Warning alert (P2)
            Alert->>Slack: Send to #alerts
            Alert->>DevTeam: Email notification
        else Info alert (P3)
            Alert->>Slack: Send to #monitoring
        end

        Note over DevTeam: Response Actions
        DevTeam->>K8s: Investigate issue
        DevTeam->>Grafana: Check dashboards
        DevTeam->>K8s: Apply fixes if needed

        alt Issue resolved
            K8s->>Prom: Metrics return to normal
            Prom->>Alert: Resolve alert
            Alert->>Slack: Send resolution notification
            Alert->>PagerDuty: Close incident
        else Issue persists
            DevTeam->>K8s: Trigger rollback
            DevTeam->>Alert: Escalate to senior team
        end
    end
```

## 6. Flux de Rollback et Recovery

```mermaid
sequenceDiagram
    participant Mon as Monitoring
    participant Alert as Alertmanager
    participant ArgoCD as ArgoCD
    participant K8s as Kubernetes
    participant Helm as Helm
    participant DB as Database
    participant DevTeam as Dev Team
    participant Backup as Backup Service

    Note over Mon: Failure Detection
    Mon->>Mon: Detect deployment failure
    Mon->>Alert: Trigger critical alert
    Alert->>DevTeam: Notify on-call engineer

    Note over DevTeam: Incident Response
    DevTeam->>ArgoCD: Check deployment status
    DevTeam->>K8s: Verify pod health
    DevTeam->>Mon: Analyze metrics/logs

    alt Manual rollback decision
        DevTeam->>ArgoCD: Initiate manual rollback
    else Automatic rollback triggered
        Mon->>ArgoCD: Trigger auto-rollback
    end

    Note over ArgoCD: Rollback Process
    ArgoCD->>ArgoCD: Identify previous stable version
    ArgoCD->>Helm: Generate rollback manifests
    Helm-->>ArgoCD: Previous version manifests

    Note over ArgoCD: Database Considerations
    alt Database migration required
        ArgoCD->>DevTeam: Request manual intervention
        DevTeam->>Backup: Restore database backup
        Backup->>DB: Restore to previous state
        DB-->>DevTeam: Database restored
        DevTeam->>ArgoCD: Continue rollback
    else No DB changes
        ArgoCD->>K8s: Apply rollback manifests
    end

    Note over K8s: Rollback Execution
    K8s->>K8s: Scale down failed version
    K8s->>K8s: Scale up previous version
    K8s->>K8s: Update service endpoints

    Note over K8s: Health Verification
    K8s->>K8s: Run health checks
    K8s->>K8s: Verify service connectivity

    alt Rollback successful
        K8s-->>ArgoCD: Rollback completed
        ArgoCD->>Mon: Update deployment status
        Mon->>Alert: Resolve incident
        Alert->>DevTeam: Rollback successful

        Note over DevTeam: Post-Incident
        DevTeam->>DevTeam: Create incident report
        DevTeam->>DevTeam: Schedule post-mortem
        DevTeam->>ArgoCD: Plan forward fix
    else Rollback failed
        K8s-->>ArgoCD: Rollback failed
        ArgoCD->>Alert: Escalate incident
        Alert->>DevTeam: Critical escalation

        Note over DevTeam: Emergency Response
        DevTeam->>K8s: Manual intervention
        DevTeam->>Backup: Full system restore
        DevTeam->>Alert: Engage incident commander
    end
```

## Légende des Acteurs

| Acteur | Description |
|--------|-------------|
| **Developer** | Développeur qui pousse le code |
| **GitHub** | Plateforme Git et déclencheur de workflow |
| **GitHub Actions** | Moteur d'exécution des workflows |
| **Docker Registry** | Registre d'images Docker (GHCR + DockerHub) |
| **ArgoCD** | Outil GitOps pour le déploiement |
| **Kubernetes** | Orchestrateur de conteneurs |
| **GPU Runner** | Runner CI/CD avec GPU |
| **GPU Node** | Nœud Kubernetes avec GPU |
| **Prometheus** | Système de monitoring |
| **Alertmanager** | Gestionnaire d'alertes |
| **Grafana** | Dashboards de visualisation |

## Notes Techniques

### Spécificités GPU
- Les services IA nécessitent des runners avec GPU NVIDIA
- Images Docker basées sur CUDA runtime
- Tests de performance obligatoires avant déploiement
- Monitoring spécifique des métriques GPU

### Sécurité
- Scans de sécurité à chaque étape
- Validation des images Docker
- Secrets gérés via Azure Key Vault
- Accès restreint aux environnements de production

### Résilience
- Rollback automatique en cas d'échec
- Health checks complets
- Monitoring proactif
- Alerting multi-canal (Slack, PagerDuty, Email)

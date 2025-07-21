# Architecture des Repositories - Projet Visiobook

Ce document définit l'organisation des repositories GitHub pour le projet Visiobook, en suivant une approche multi-repo avec les meilleures pratiques et conventions de nommage modernes.

## Vue d'ensemble de l'organisation

```
GitHub Organization: visiobook-esp
├── Core Services (5 repositories)
├── AI Services (3 repositories)
├── Support Services (6 repositories)
├── Content & Infrastructure (3 repositories)
├── Applications (3 repositories)
└── Shared & Documentation (2 repositories)

Total: 22 repositories
```

## Convention de nommage

### Règles générales
- **Kebab-case** pour tous les noms de repositories
- **Préfixes** pour catégoriser les services
- **Suffixes** pour identifier le type de repository
- **Pas d'abréviations** ambiguës
- **Noms descriptifs** et explicites

### Préfixes par catégorie
- `core-` : Services métier principaux
- `ai-` : Services d'intelligence artificielle
- `support-` : Services de support et utilitaires
- `infra-` : Infrastructure et déploiement
- `web-` : Applications web frontend
- `mobile-` : Applications mobiles
- `shared-` : Bibliothèques partagées
- `docs-` : Documentation

### Suffixes par type
- `-service` : Microservices backend
- `-app` : Applications frontend
- `-lib` : Bibliothèques partagées

## 1. Core Services (5 repositories)

Services métier principaux de l'application Visiobook.

```
visiobook-esp/
├── core-user-service
├── core-project-service
├── core-payment-service
├── core-notification-service
└── core-api-gateway
```

### Structure type : `core-user-service`

```
core-user-service/
├── .github/
│   ├── workflows/
│   │   ├── ci-cd.yml
│   │   ├── security-scan.yml
│   │   ├── dependency-update.yml
│   │   └── release.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   ├── feature_request.md
│   │   └── security_issue.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── CODEOWNERS
├── src/
│   ├── [Structure adaptée selon la stack choisie]
│   │   ├── controllers/ ou routes/ ou handlers/
│   │   ├── services/ ou business/
│   │   ├── repositories/ ou data/
│   │   ├── models/ ou entities/
│   │   ├── dto/ ou schemas/
│   │   ├── config/
│   │   └── [Fichier principal d'application]
│   └── resources/ ou config/ ou assets/
│       ├── [Fichiers de configuration par environnement]
│       └── [Migrations de base de données si applicable]
├── tests/ ou test/
│   ├── unit/
│   ├── integration/
│   └── [Structure selon framework de test choisi]
├── docs/
│   ├── api/
│   │   ├── openapi.yml
│   │   ├── postman-collection.json
│   │   └── api-examples.md
│   ├── architecture/
│   │   ├── service-design.md
│   │   ├── database-schema.md
│   │   └── security-model.md
│   └── deployment/
│       ├── deployment-guide.md
│       └── troubleshooting.md
├── scripts/
│   ├── build.sh
│   ├── test.sh
│   ├── deploy.sh
│   └── db-migrate.sh
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-dev.yaml
│   ├── values-staging.yaml
│   ├── values-prod.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── configmap.yaml
│       ├── secret.yaml
│       ├── hpa.yaml
│       └── servicemonitor.yaml
├── Dockerfile
├── Dockerfile.dev
├── docker-compose.yml
├── docker-compose.test.yml
├── [Fichier de configuration du projet selon la stack]
│   └── (pom.xml, package.json, go.mod, requirements.txt, etc.)
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
└── .gitignore
```

## 2. AI Services (3 repositories)

Services spécialisés dans l'intelligence artificielle et le traitement multimédia.

```
visiobook-esp/
├── ai-analysis-service
├── ai-media-generation-service
└── ai-storyboard-assembly-service
```

### Structure type : `ai-analysis-service`

```
ai-analysis-service/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml
│       ├── model-validation.yml
│       ├── gpu-tests.yml
│       └── performance-benchmarks.yml
├── src/
│   ├── api/
│   │   ├── routes/
│   │   │   ├── analysis.py
│   │   │   ├── health.py
│   │   │   └── metrics.py
│   │   ├── middleware/
│   │   │   ├── auth.py
│   │   │   ├── logging.py
│   │   │   └── rate_limiting.py
│   │   └── app.py
│   ├── models/
│   │   ├── semantic_analysis/
│   │   │   ├── bert_model.py
│   │   │   └── transformer_model.py
│   │   ├── scene_extraction/
│   │   │   ├── scene_detector.py
│   │   │   └── entity_extractor.py
│   │   └── summarization/
│   │       ├── abstractive_summarizer.py
│   │       └── extractive_summarizer.py
│   ├── services/
│   │   ├── analysis_service.py
│   │   ├── model_loader.py
│   │   ├── preprocessing.py
│   │   └── postprocessing.py
│   ├── utils/
│   │   ├── text_processing.py
│   │   ├── validation.py
│   │   └── gpu_utils.py
│   └── config/
│       ├── model_config.py
│       ├── settings.py
│       └── logging_config.py
├── models/
│   ├── checkpoints/
│   │   ├── bert-base-uncased/
│   │   └── custom-scene-extractor/
│   ├── configs/
│   │   ├── bert_config.json
│   │   └── scene_config.json
│   └── weights/
│       └── .gitkeep
├── data/
│   ├── training/
│   │   └── .gitkeep
│   ├── validation/
│   │   └── .gitkeep
│   └── test/
│       ├── sample_texts.json
│       └── expected_outputs.json
├── notebooks/
│   ├── exploration/
│   │   ├── data_analysis.ipynb
│   │   └── model_comparison.ipynb
│   ├── training/
│   │   ├── train_scene_extractor.ipynb
│   │   └── fine_tune_bert.ipynb
│   └── evaluation/
│       ├── model_evaluation.ipynb
│       └── performance_analysis.ipynb
├── tests/
│   ├── unit/
│   │   ├── test_analysis_service.py
│   │   ├── test_models.py
│   │   └── test_utils.py
│   ├── integration/
│   │   ├── test_api_endpoints.py
│   │   └── test_model_pipeline.py
│   └── performance/
│       ├── test_gpu_performance.py
│       └── test_memory_usage.py
├── docs/
│   ├── model-documentation.md
│   ├── api-reference.md
│   ├── training-guide.md
│   └── performance-benchmarks.md
├── requirements/
│   ├── base.txt
│   ├── dev.txt
│   ├── prod.txt
│   └── gpu.txt
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── gpu-deployment.yaml
│       └── model-configmap.yaml
├── Dockerfile
├── Dockerfile.gpu
├── docker-compose.yml
├── docker-compose.gpu.yml
├── pyproject.toml
├── README.md
└── .gitignore
```

## 3. Support Services (6 repositories)

Services de support, monitoring et utilitaires.

```
visiobook-esp/
├── support-config-service
├── support-monitoring-service
├── support-logging-service
├── support-security-service
├── support-analytics-service
└── support-storage-service
```

### Structure type : `support-monitoring-service`

```
support-monitoring-service/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml
│       └── monitoring-tests.yml
├── src/
│   ├── collectors/
│   │   ├── [Collecteur de métriques]
│   │   ├── [Collecteur de logs]
│   │   └── [Collecteur de traces]
│   ├── processors/
│   │   ├── [Agrégateur]
│   │   └── [Gestionnaire d'alertes]
│   ├── exporters/
│   │   ├── [Exporteur Prometheus]
│   │   └── [Exporteur Grafana]
│   ├── config/
│   │   └── [Configuration]
│   └── [Fichier principal d'application]
├── configs/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── alert_rules.yml
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   ├── services-overview.json
│   │   │   ├── ai-services.json
│   │   │   └── infrastructure.json
│   │   └── datasources/
│   │       └── prometheus.yml
│   └── alertmanager/
│       └── alertmanager.yml
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── prometheus/
│       ├── grafana/
│       └── alertmanager/
├── docs/
│   ├── monitoring-guide.md
│   ├── alerting-runbook.md
│   └── dashboard-guide.md
├── [Fichiers de configuration selon la stack choisie]
├── Dockerfile
├── README.md
└── .gitignore
```

## 4. Content & Infrastructure (3 repositories)

Ingestion de contenu et infrastructure as code.

```
visiobook-esp/
├── content-ingestion-service
├── infra-bicep-azure
└── infra-helm-charts
```

### Structure détaillée : `infra-bicep-azure`

```
infra-bicep-azure/
├── .github/
│   └── workflows/
│       ├── bicep-validate.yml
│       ├── bicep-deploy-dev.yml
│       ├── bicep-deploy-staging.yml
│       ├── bicep-deploy-prod.yml
│       └── bicep-destroy.yml
├── modules/
│   ├── aks/
│   │   ├── main.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── networking/
│   │   ├── vnet.bicep
│   │   ├── nsg.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── databases/
│   │   ├── postgresql.bicep
│   │   ├── cosmosdb.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── storage/
│   │   ├── blob-storage.bicep
│   │   ├── file-storage.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── messaging/
│   │   ├── service-bus.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── security/
│   │   ├── key-vault.bicep
│   │   ├── managed-identity.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   ├── monitoring/
│   │   ├── log-analytics.bicep
│   │   ├── application-insights.bicep
│   │   ├── parameters.json
│   │   └── README.md
│   └── container-registry/
│       ├── acr.bicep
│       ├── parameters.json
│       └── README.md
├── environments/
│   ├── dev/
│   │   ├── main.bicep
│   │   ├── parameters.json
│   │   └── deploy.sh
│   ├── staging/
│   │   ├── main.bicep
│   │   ├── parameters.json
│   │   └── deploy.sh
│   └── prod/
│       ├── main.bicep
│       ├── parameters.json
│       └── deploy.sh
├── scripts/
│   ├── bootstrap.sh
│   ├── validate-all.sh
│   ├── deploy-env.sh
│   ├── destroy-env.sh
│   └── migrate-resources.sh
├── docs/
│   ├── infrastructure-overview.md
│   ├── deployment-guide.md
│   ├── module-documentation.md
│   ├── troubleshooting.md
│   └── cost-optimization.md
├── tests/
│   ├── unit/
│   │   └── bicep-tests.json
│   └── integration/
│       └── deployment-tests.sh
├── .bicepconfig.json
├── README.md
├── CHANGELOG.md
└── .gitignore
```

#### Exemple de module AKS : `modules/aks/main.bicep`

```bicep
@description('The name of the AKS cluster')
param clusterName string

@description('The location for the AKS cluster')
param location string = resourceGroup().location

@description('The Kubernetes version')
param kubernetesVersion string = '1.28.0'

@description('The node count for the default node pool')
param nodeCount int = 3

@description('The VM size for the default node pool')
param nodeVmSize string = 'Standard_D4s_v3'

@description('The VM size for the GPU node pool')
param gpuNodeVmSize string = 'Standard_NC6s_v3'

@description('The node count for the GPU node pool')
param gpuNodeCount int = 1

@description('The subnet ID for the AKS cluster')
param subnetId string

@description('The managed identity ID for the AKS cluster')
param managedIdentityId string

resource aksCluster 'Microsoft.ContainerService/managedClusters@2023-08-01' = {
  name: clusterName
  location: location
  identity: {
    type: 'UserAssigned'
    userAssignedIdentities: {
      '${managedIdentityId}': {}
    }
  }
  properties: {
    kubernetesVersion: kubernetesVersion
    dnsPrefix: '${clusterName}-dns'
    agentPoolProfiles: [
      {
        name: 'default'
        count: nodeCount
        vmSize: nodeVmSize
        osType: 'Linux'
        mode: 'System'
        vnetSubnetID: subnetId
        enableAutoScaling: true
        minCount: 1
        maxCount: 10
      }
      {
        name: 'gpu'
        count: gpuNodeCount
        vmSize: gpuNodeVmSize
        osType: 'Linux'
        mode: 'User'
        vnetSubnetID: subnetId
        enableAutoScaling: true
        minCount: 0
        maxCount: 5
        nodeLabels: {
          'workload-type': 'gpu'
        }
        nodeTaints: [
          'nvidia.com/gpu=true:NoSchedule'
        ]
      }
    ]
    networkProfile: {
      networkPlugin: 'azure'
      networkPolicy: 'azure'
      serviceCidr: '10.0.0.0/16'
      dnsServiceIP: '10.0.0.10'
    }
    addonProfiles: {
      azureKeyvaultSecretsProvider: {
        enabled: true
      }
      azurepolicy: {
        enabled: true
      }
      omsagent: {
        enabled: true
        config: {
          logAnalyticsWorkspaceResourceID: '/subscriptions/${subscription().subscriptionId}/resourceGroups/${resourceGroup().name}/providers/Microsoft.OperationalInsights/workspaces/${clusterName}-logs'
        }
      }
    }
  }
}

output clusterName string = aksCluster.name
output clusterFqdn string = aksCluster.properties.fqdn
output kubeletIdentityObjectId string = aksCluster.properties.identityProfile.kubeletidentity.objectId
```

### Structure détaillée : `infra-helm-charts`

```
infra-helm-charts/
├── .github/
│   └── workflows/
│       ├── lint-charts.yml
│       ├── test-charts.yml
│       ├── release-charts.yml
│       └── security-scan.yml
├── charts/
│   ├── visiobook-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── templates/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── configmap.yaml
│   │   │   ├── secret.yaml
│   │   │   ├── hpa.yaml
│   │   │   ├── pdb.yaml
│   │   │   ├── servicemonitor.yaml
│   │   │   ├── ingress.yaml
│   │   │   └── networkpolicy.yaml
│   │   └── values/
│   │       ├── core-user-service.yaml
│   │       ├── core-project-service.yaml
│   │       ├── ai-analysis-service.yaml
│   │       ├── ai-media-generation-service.yaml
│   │       └── support-monitoring-service.yaml
│   ├── visiobook-ai-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── gpu-deployment.yaml
│   │       ├── model-configmap.yaml
│   │       ├── gpu-service.yaml
│   │       └── resource-quota.yaml
│   ├── visiobook-database/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── postgresql.yaml
│   │       ├── cosmosdb-secret.yaml
│   │       └── backup-cronjob.yaml
│   └── visiobook-monitoring/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── prometheus/
│           ├── grafana/
│           └── alertmanager/
├── scripts/
│   ├── lint-all.sh
│   ├── test-all.sh
│   ├── package-all.sh
│   └── deploy-chart.sh
├── docs/
│   ├── chart-development.md
│   ├── values-reference.md
│   └── deployment-patterns.md
├── tests/
│   ├── unit/
│   └── integration/
└── README.md
```

## 5. Applications (3 repositories)

Applications frontend web et mobile.

```
visiobook-esp/
├── web-user-portal
├── web-admin-dashboard
└── mobile-react-native-app
```

### Structure type : `web-user-portal`

```
web-user-portal/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml
│       ├── lighthouse.yml
│       ├── e2e-tests.yml
│       └── security-scan.yml
├── public/
│   ├── index.html
│   ├── manifest.json
│   ├── robots.txt
│   └── assets/
│       ├── icons/
│       ├── images/
│       └── fonts/
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   ├── Modal/
│   │   │   └── Loading/
│   │   ├── layout/
│   │   │   ├── Header/
│   │   │   ├── Sidebar/
│   │   │   ├── Footer/
│   │   │   └── Layout/
│   │   └── features/
│   │       ├── auth/
│   │       ├── projects/
│   │       ├── dashboard/
│   │       └── profile/
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── RegisterPage.tsx
│   │   │   └── ForgotPasswordPage.tsx
│   │   ├── dashboard/
│   │   │   └── DashboardPage.tsx
│   │   ├── projects/
│   │   │   ├── ProjectsPage.tsx
│   │   │   ├── ProjectDetailPage.tsx
│   │   │   └── CreateProjectPage.tsx
│   │   └── profile/
│   │       └── ProfilePage.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useApi.ts
│   │   ├── useLocalStorage.ts
│   │   └── useWebSocket.ts
│   ├── services/
│   │   ├── api/
│   │   │   ├── authApi.ts
│   │   │   ├── projectsApi.ts
│   │   │   ├── userApi.ts
│   │   │   └── baseApi.ts
│   │   ├── auth/
│   │   │   ├── authService.ts
│   │   │   └── tokenService.ts
│   │   └── websocket/
│   │       └── websocketService.ts
│   ├── store/
│   │   ├── slices/
│   │   │   ├── authSlice.ts
│   │   │   ├── projectsSlice.ts
│   │   │   └── uiSlice.ts
│   │   ├── middleware/
│   │   │   └── apiMiddleware.ts
│   │   └── index.ts
│   ├── utils/
│   │   ├── constants.ts
│   │   ├── helpers.ts
│   │   ├── validators.ts
│   │   └── formatters.ts
│   ├── types/
│   │   ├── auth.ts
│   │   ├── projects.ts
│   │   ├── user.ts
│   │   └── api.ts
│   ├── styles/
│   │   ├── globals.css
│   │   ├── variables.css
│   │   └── components/
│   │       ├── button.css
│   │       └── modal.css
│   ├── App.tsx
│   ├── index.tsx
│   └── setupTests.ts
├── tests/
│   ├── unit/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── utils/
│   ├── integration/
│   │   └── api/
│   └── e2e/
│       ├── auth.spec.ts
│       ├── projects.spec.ts
│       └── dashboard.spec.ts
├── docs/
│   ├── component-library.md
│   ├── development-guide.md
│   ├── deployment-guide.md
│   └── testing-guide.md
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       └── configmap.yaml
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
├── eslint.config.js
├── prettier.config.js
├── Dockerfile
├── Dockerfile.dev
├── nginx.conf
├── docker-compose.yml
├── README.md
└── .gitignore
```

## 6. Shared & Documentation (2 repositories)

Bibliothèques partagées et documentation centralisée.

```
visiobook-esp/
├── shared-libraries
└── docs-architecture
```

### Structure : `shared-libraries`

```
shared-libraries/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml
│       ├── publish-npm.yml
│       └── publish-maven.yml
├── packages/
│   ├── typescript/
│   │   ├── common-types/
│   │   ├── api-client/
│   │   ├── ui-components/
│   │   └── utils/
│   ├── java/
│   │   ├── common-models/
│   │   ├── security-utils/
│   │   └── monitoring-utils/
│   └── python/
│       ├── ai-utils/
│       ├── data-processing/
│       └── model-commons/
├── docs/
│   ├── typescript-libs.md
│   ├── java-libs.md
│   └── python-libs.md
├── scripts/
│   ├── build-all.sh
│   ├── test-all.sh
│   └── publish-all.sh
├── lerna.json
├── package.json
└── README.md
```

## Conventions de branches

### Stratégie Git Flow adaptée

#### Branches principales
- `main` : Code de production stable
- `develop` : Branche de développement principal
- `staging` : Code en cours de validation

#### Branches de fonctionnalités
- `feature/VIS-123-user-authentication`
- `feature/VIS-124-ai-model-integration`
- `bugfix/VIS-125-payment-validation`
- `hotfix/VIS-126-security-patch`

#### Convention de nommage des branches
```
<type>/<ticket-id>-<description>

Types:
- feature/ : Nouvelles fonctionnalités
- bugfix/ : Corrections de bugs
- hotfix/ : Corrections urgentes
- refactor/ : Refactoring de code
- docs/ : Documentation
- chore/ : Tâches de maintenance
```

## Conventions de commits

### Format Conventional Commits

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Types de commits
- `feat` : Nouvelle fonctionnalité
- `fix` : Correction de bug
- `docs` : Documentation
- `style` : Formatage, point-virgules manquants, etc.
- `refactor` : Refactoring de code
- `perf` : Amélioration des performances
- `test` : Ajout ou modification de tests
- `chore` : Maintenance, mise à jour des dépendances
- `ci` : Modifications CI/CD
- `build` : Modifications du système de build

#### Exemples
```bash
feat(auth): add OAuth2 integration
fix(payment): resolve transaction timeout issue
docs(api): update OpenAPI specifications
refactor(ai): optimize model loading performance
test(user): add integration tests for user service
ci(github): update workflow for security scanning
```

## GitHub Actions Workflows

### Workflow type pour Core Services

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  AZURE_RESOURCE_GROUP: visiobook-rg
  AKS_CLUSTER: visiobook-aks

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup runtime environment
        uses: ./.github/actions/setup-runtime
        # Action composite qui détecte et configure l'environnement
        # selon les fichiers présents (pom.xml, package.json, go.mod, etc.)

      - name: Install dependencies
        run: ./scripts/install-deps.sh

      - name: Run tests
        run: ./scripts/test.sh

      - name: Generate test report
        uses: dorny/test-reporter@v1
        if: success() || failure()
        with:
          name: Tests
          path: test-results/**/*.xml
          reporter: java-junit

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run security scan
        run: ./scripts/security-scan.sh
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup runtime environment
        uses: ./.github/actions/setup-runtime

      - name: Build application
        run: ./scripts/build.sh

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Get AKS credentials
        run: |
          az aks get-credentials --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --name ${{ env.AKS_CLUSTER }}

      - name: Deploy to staging
        run: |
          helm upgrade --install ${{ github.event.repository.name }} ./helm \
            --namespace staging \
            --create-namespace \
            --values ./helm/values-staging.yaml \
            --set image.tag=${{ github.sha }}

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Get AKS credentials
        run: |
          az aks get-credentials --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --name ${{ env.AKS_CLUSTER }}

      - name: Deploy to production
        run: |
          helm upgrade --install ${{ github.event.repository.name }} ./helm \
            --namespace production \
            --create-namespace \
            --values ./helm/values-prod.yaml \
            --set image.tag=${{ github.sha }}
```

### Workflow type pour AI Services

```yaml
# .github/workflows/ci-cd.yml
name: AI Service CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  AZURE_RESOURCE_GROUP: visiobook-rg
  AKS_CLUSTER: visiobook-aks

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Cache pip dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements/*.txt') }}

      - name: Install dependencies
        run: |
          pip install -r requirements/dev.txt

      - name: Run unit tests
        run: |
          pytest tests/unit/ --cov=src --cov-report=xml

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  gpu-test:
    runs-on: [self-hosted, gpu]
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install GPU dependencies
        run: |
          pip install -r requirements/gpu.txt

      - name: Run GPU tests
        run: |
          pytest tests/performance/ --gpu

      - name: Model validation
        run: |
          python scripts/validate_models.py

  build:
    needs: [test, gpu-test]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        variant: [cpu, gpu]
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: Dockerfile.${{ matrix.variant }}
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}-${{ matrix.variant }}
```

### Workflow type pour Infrastructure Bicep

```yaml
# .github/workflows/bicep-validate.yml
name: Bicep Validation

on:
  push:
    branches: [ main, develop ]
    paths: ['modules/**', 'environments/**']
  pull_request:
    branches: [ main, develop ]
    paths: ['modules/**', 'environments/**']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Validate Bicep files
        run: |
          for file in $(find . -name "*.bicep"); do
            echo "Validating $file"
            az bicep build --file $file
          done

      - name: Run Bicep linter
        run: |
          az bicep lint --file environments/dev/main.bicep
          az bicep lint --file environments/staging/main.bicep
          az bicep lint --file environments/prod/main.bicep

      - name: What-if deployment (dev)
        if: github.event_name == 'pull_request'
        run: |
          az deployment group what-if \
            --resource-group visiobook-dev-rg \
            --template-file environments/dev/main.bicep \
            --parameters @environments/dev/parameters.json
```

## Gestion des versions

### Semantic Versioning (SemVer)

Format: `MAJOR.MINOR.PATCH`

- **MAJOR** : Changements incompatibles de l'API
- **MINOR** : Nouvelles fonctionnalités compatibles
- **PATCH** : Corrections de bugs compatibles

#### Exemples
- `1.0.0` : Version initiale
- `1.1.0` : Nouvelle fonctionnalité
- `1.1.1` : Correction de bug
- `2.0.0` : Changement majeur incompatible

### Tags et releases

```bash
# Création d'un tag
git tag -a v1.2.0 -m "Release version 1.2.0"

# Convention de nommage des tags
v<version>
v1.0.0
v1.1.0-beta.1
v2.0.0-rc.1
```

## GitHub Teams et permissions

### Organisation des équipes

```
visiobook-esp/
├── @visiobook-esp/admins
│   ├── Admin access to all repositories
│   └── Can manage organization settings
├── @visiobook-esp/core-team
│   ├── Write access to core-* repositories
│   ├── Review required for main branch
│   └── Can approve production deployments
├── @visiobook-esp/ai-team
│   ├── Write access to ai-* repositories
│   ├── Access to GPU runners
│   └── Specialized in ML/AI development
├── @visiobook-esp/frontend-team
│   ├── Write access to web-* repositories
│   ├── Access to mobile-* repositories
│   └── UI/UX focused development
├── @visiobook-esp/devops-team
│   ├── Write access to infra-* repositories
│   ├── Admin access to support-* repositories
│   └── CI/CD and deployment management
├── @visiobook-esp/support-team
│   ├── Write access to support-* repositories
│   └── Monitoring and infrastructure support
└── @visiobook-esp/contractors
    ├── Limited access based on project
    └── Time-limited permissions
```

### Protection des branches

#### Configuration type pour repositories critiques

```yaml
# Branch protection rules
main:
  required_status_checks:
    strict: true
    contexts:
      - "test"
      - "security-scan"
      - "build"
  enforce_admins: false
  required_pull_request_reviews:
    required_approving_review_count: 2
    dismiss_stale_reviews: true
    require_code_owner_reviews: true
    restrict_pushes: true
  restrictions:
    users: []
    teams:
      - "core-team"
      - "admins"
  allow_force_pushes: false
  allow_deletions: false

develop:
  required_status_checks:
    strict: true
    contexts:
      - "test"
      - "security-scan"
  required_pull_request_reviews:
    required_approving_review_count: 1
    dismiss_stale_reviews: true
```

### CODEOWNERS par repository

#### Exemple pour `core-user-service`

```
# CODEOWNERS
* @visiobook-esp/core-team
/src/main/java/com/visiobook/user/security/ @visiobook-esp/security-team
/helm/ @visiobook-esp/devops-team
/docs/ @visiobook-esp/core-team @visiobook-esp/tech-writers
/.github/ @visiobook-esp/devops-team
/Dockerfile* @visiobook-esp/devops-team
```

#### Exemple pour `ai-analysis-service`

```
# CODEOWNERS
* @visiobook-esp/ai-team
/src/models/ @visiobook-esp/ai-team @visiobook-esp/ml-engineers
/notebooks/ @visiobook-esp/data-scientists
/helm/ @visiobook-esp/devops-team
/requirements/gpu.txt @visiobook-esp/ai-team @visiobook-esp/devops-team
```

## Sécurité et secrets

### Gestion des secrets GitHub

#### Secrets au niveau organisation
- `AZURE_CLIENT_ID` : ID client Azure pour OIDC
- `AZURE_TENANT_ID` : ID tenant Azure
- `AZURE_SUBSCRIPTION_ID` : ID subscription Azure
- `SNYK_TOKEN` : Token pour scans de sécurité
- `SONAR_TOKEN` : Token SonarCloud

#### Secrets par environnement
```
Environments:
├── development
│   ├── AZURE_RESOURCE_GROUP: visiobook-dev-rg
│   └── DATABASE_CONNECTION_STRING: ***
├── staging
│   ├── AZURE_RESOURCE_GROUP: visiobook-staging-rg
│   └── DATABASE_CONNECTION_STRING: ***
└── production
    ├── AZURE_RESOURCE_GROUP: visiobook-prod-rg
    ├── DATABASE_CONNECTION_STRING: ***
    └── Required reviewers: @visiobook-esp/admins
```

### Scans de sécurité automatisés

#### CodeQL Analysis
```yaml
# .github/workflows/codeql-analysis.yml
name: "CodeQL"

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * 1'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'java', 'javascript', 'python' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}

    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
```

## Monitoring et observabilité

### Métriques par repository

#### GitHub Actions metrics
- Build success rate
- Deployment frequency
- Lead time for changes
- Mean time to recovery
- Pull request metrics

#### Code quality metrics
- Code coverage
- Technical debt ratio
- Security vulnerabilities
- Dependency freshness

### Dashboards recommandés

#### 1. Repository Health Dashboard
```
Métriques:
├── Commit frequency par repository
├── PR merge rate et temps de review
├── Issue resolution time
├── Code review participation
├── Branch protection compliance
└── Security scan results
```

#### 2. Deployment Dashboard
```
Métriques:
├── Deployment frequency par service
├── Deployment success rate
├── Rollback frequency
├── Lead time for changes
├── Recovery time
└── Environment health status
```

#### 3. Security Dashboard
```
Métriques:
├── Vulnerability scans results
├── Dependency alerts status
├── Secret scanning alerts
├── Code scanning results
├── Branch protection violations
└── Access review compliance
```

## Automatisation et outils

### GitHub Apps recommandées

1. **Dependabot** : Mise à jour automatique des dépendances
   ```yaml
   # .github/dependabot.yml
   version: 2
   updates:
     - package-ecosystem: "maven"
       directory: "/"
       schedule:
         interval: "weekly"
       reviewers:
         - "visiobook-esp/core-team"
   ```

2. **CodeQL** : Analyse de sécurité du code
3. **Snyk** : Scan de vulnérabilités
4. **SonarCloud** : Qualité du code
5. **Renovate** : Gestion avancée des dépendances

### Pre-commit hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-merge-conflict

  - repo: https://github.com/psf/black
    rev: 22.3.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]
```

## Template README.md standard

```markdown
# Service Name

Brief description of the service and its purpose in the Visiobook ecosystem.

## 🚀 Quick Start

### Prerequisites
- Java 17+ / Python 3.11+ / Node.js 18+
- Docker & Docker Compose
- Access to Azure resources

### Local Development
```bash
# Clone the repository
git clone https://github.com/visiobook-esp/service-name.git
cd service-name

# Install dependencies
./scripts/setup.sh

# Start development environment
docker-compose up -d

# Run the service
./scripts/dev.sh
```

## 🏗️ Architecture

- **Technology Stack**: [Java Spring Boot / Python FastAPI / React TypeScript]
- **Database**: [PostgreSQL / CosmosDB]
- **Message Queue**: Azure Service Bus
- **Monitoring**: Prometheus + Grafana
- **Deployment**: Kubernetes + Helm

## 📖 Documentation

- [API Documentation](docs/api/) - OpenAPI specifications
- [Architecture](docs/architecture/) - Service design and patterns
- [Deployment Guide](docs/deployment/) - How to deploy and configure
- [Troubleshooting](docs/troubleshooting.md) - Common issues and solutions

## 🧪 Testing

```bash
# Unit tests
./scripts/test.sh

# Integration tests
./scripts/test-integration.sh

# Performance tests (AI services only)
./scripts/test-performance.sh
```

## 🚀 Deployment

### Local Testing
```bash
docker-compose -f docker-compose.test.yml up
```

### Kubernetes
```bash
# Deploy to staging
helm upgrade --install service-name ./helm \
  --namespace staging \
  --values ./helm/values-staging.yaml

# Deploy to production
helm upgrade --install service-name ./helm \
  --namespace production \
  --values ./helm/values-prod.yaml
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/VIS-123-new-feature`)
3. Commit your changes (`git commit -m 'feat: add new feature'`)
4. Push to the branch (`git push origin feature/VIS-123-new-feature`)
5. Open a Pull Request

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## 📊 Monitoring

- **Health Check**: `/health`
- **Metrics**: `/metrics` (Prometheus format)
- **Grafana Dashboard**: [Link to dashboard]
- **Logs**: Available in Azure Log Analytics

## 🔒 Security

- Security scans run automatically on every PR
- Secrets are managed via Azure Key Vault
- All dependencies are regularly updated via Dependabot

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file.

## 📞 Support

- **Documentation**: [docs.visiobook.com](https://docs.visiobook.com)
- **Issues**: [GitHub Issues](https://github.com/visiobook-esp/service-name/issues)
- **Slack**: #visiobook-dev
- **On-call**: PagerDuty integration for production issues
```

## Migration et évolution

### Plan de migration vers cette architecture

#### Phase 1 : Préparation (Semaine 1-2)
1. Création de l'organisation GitHub `visiobook-esp`
2. Configuration des équipes et permissions
3. Mise en place des templates de repositories
4. Configuration des secrets et environnements

#### Phase 2 : Infrastructure (Semaine 3-4)
1. Création du repository `infra-bicep-azure`
2. Développement des modules Bicep
3. Déploiement des environnements dev/staging
4. Création du repository `infra-helm-charts`

#### Phase 3 : Services Core (Semaine 5-8)
1. Migration des 5 services core
2. Configuration des workflows CI/CD
3. Tests et validation des déploiements
4. Documentation des APIs

#### Phase 4 : Services AI (Semaine 9-12)
1. Migration des 3 services AI
2. Configuration des runners GPU
3. Tests de performance et validation des modèles
4. Optimisation des workflows

#### Phase 5 : Applications et Support (Semaine 13-16)
1. Migration des applications frontend
2. Migration des services support
3. Configuration du monitoring complet
4. Tests end-to-end

#### Phase 6 : Finalisation (Semaine 17-18)
1. Migration des bibliothèques partagées
2. Documentation complète
3. Formation des équipes
4. Go-live production

### Évolution continue

#### Reviews trimestrielles
- Architecture des repositories
- Performance des workflows CI/CD
- Sécurité et compliance
- Coûts d'infrastructure

#### Métriques de succès
- Temps de déploiement < 10 minutes
- Taux de succès des builds > 95%
- Temps de résolution des incidents < 2h
- Couverture de code > 80%

## Conclusion

Cette architecture multi-repo pour le projet Visiobook offre :

### ✅ Avantages
1. **Scalabilité** : Chaque service évolue indépendamment
2. **Sécurité** : Permissions granulaires par équipe et service
3. **Performance** : Builds parallèles et optimisés
4. **Maintenabilité** : Structure claire et conventions cohérentes
5. **Observabilité** : Monitoring complet à tous les niveaux
6. **Automatisation** : CI/CD intégré avec Azure et Kubernetes
7. **Collaboration** : Workflows optimisés pour les équipes distribuées

### 🎯 Objectifs atteints
- **22 repositories** organisés logiquement
- **Infrastructure as Code** complète avec Azure Bicep
- **CI/CD moderne** avec GitHub Actions
- **Sécurité intégrée** à tous les niveaux
- **Monitoring proactif** et alerting
- **Documentation vivante** et maintenue

Cette organisation facilite le développement, le déploiement et la maintenance du projet Visiobook tout en respectant les standards de l'industrie et en optimisant l'utilisation de GitHub Teams.

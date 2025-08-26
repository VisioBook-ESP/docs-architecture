# Definition of Ready (DoR) - Microservices Visiobook

## Vue d'ensemble

Cette Definition of Ready établit les **critères obligatoires** qu'un microservice Visiobook doit respecter avant d'être considéré comme prêt pour le développement, les tests et le déploiement. Elle est basée sur les guidelines de développement Visiobook et l'architecture microservices prioritaire.

### Objectifs

- **Qualité** : Garantir un niveau de qualité uniforme pour tous les microservices
- **Cohérence** : Assurer la conformité aux standards Visiobook
- **Interopérabilité** : Faciliter l'intégration entre microservices
- **Déployabilité** : Permettre un déploiement sûr et automatisé
- **Observabilité** : Assurer le monitoring et le debugging efficaces

## 1. Critères Techniques Obligatoires

### ✅ 1.1 Structure de Projet Standardisée

**Critères :**
- [ ] Structure de projet conforme aux guidelines Visiobook
- [ ] Séparation claire des responsabilités (controllers, services, models)
- [ ] Configuration externalisée via variables d'environnement
- [ ] Documentation README.md complète avec instructions de setup

**Validation :**
```bash
# Vérifier la structure
ls -la src/
# Doit contenir : controllers/, services/, models/, middleware/, utils/, config/, types/

# Vérifier la documentation
grep -q "## Installation" README.md
grep -q "## Configuration" README.md
grep -q "## API Documentation" README.md
```

### ✅ 1.2 Standards de Code

**Critères :**
- [ ] Linting passé sans erreurs (ESLint/Pylint/Golint selon stack)
- [ ] Formatage uniforme appliqué (Prettier/Black/gofmt)
- [ ] Types stricts validés (TypeScript strict mode si applicable)
- [ ] Pas de logs de debug en production
- [ ] Gestion d'erreurs complète avec try-catch appropriés

**Validation :**
```bash
# Node.js/TypeScript
npm run lint
npm run type-check
npm run format:check

# Python
pylint app/
black --check app/
mypy app/

# Go
golangci-lint run
go fmt ./...
go vet ./...
```

### ✅ 1.3 Configuration et Sécurité

**Critères :**
- [ ] Variables d'environnement documentées dans .env.example
- [ ] Secrets externalisés (pas de hardcoding)
- [ ] Configuration par environnement (dev/staging/prod)
- [ ] Headers de sécurité configurés
- [ ] Validation des entrées implémentée

**Variables obligatoires :**
```env
# Service
SERVICE_NAME=nom-du-service
SERVICE_VERSION=1.0.0
NODE_ENV=production
PORT=8080
LOG_LEVEL=info

# Database
DATABASE_URL=postgresql://...
REDIS_URL=redis://...

# Security
JWT_SECRET=***
CORS_ORIGINS=https://visiobook.com

# External Services
API_GATEWAY_URL=http://api-gateway-service:8080
```

## 2. Critères de Qualité et Tests

### ✅ 2.1 Couverture de Tests

**Critères obligatoires :**
- [ ] Couverture de tests ≥ 80%
- [ ] Tests unitaires pour toute la logique métier
- [ ] Tests d'intégration pour les endpoints API
- [ ] Tests de contrat pour les dépendances externes
- [ ] Mocks configurés pour tous les services externes

**Validation :**
```bash
# Node.js
npm run test:coverage
# Vérifier : Lines > 80%, Functions > 80%, Branches > 80%

# Python
pytest --cov=app --cov-report=term-missing --cov-fail-under=80

# Go
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```

### ✅ 2.2 Types de Tests Requis

**Tests unitaires :**
- [ ] Logique métier isolée
- [ ] Services avec mocks des dépendances
- [ ] Utilitaires et helpers
- [ ] Validation des DTOs/schemas

**Tests d'intégration :**
- [ ] Endpoints API complets
- [ ] Intégration base de données
- [ ] Middleware et authentification
- [ ] Gestion d'erreurs

**Tests de contrat :**
- [ ] Validation des réponses API
- [ ] Schemas OpenAPI respectés
- [ ] Codes de statut HTTP corrects
- [ ] Headers obligatoires présents

### ✅ 2.3 Stratégie de Mocking

**Mocks obligatoires :**
- [ ] Services externes (APIs tierces)
- [ ] Base de données (pour tests unitaires)
- [ ] Services internes Visiobook
- [ ] Ressources système (fichiers, réseau)

**Configuration mocks :**
```typescript
// Exemple Node.js
// tests/mocks/external-services.mock.ts
export const mockDatabaseService = {
  getConnection: jest.fn().mockResolvedValue(mockConnection),
  executeQuery: jest.fn(),
  transaction: jest.fn(),
};

export const mockStorageService = {
  uploadFile: jest.fn().mockResolvedValue({ url: 'mock-url' }),
  deleteFile: jest.fn().mockResolvedValue(true),
};
```

## 3. Critères d'Infrastructure et Déploiement

### ✅ 3.1 Dockerfile Standardisé

**Critères :**
- [ ] Dockerfile multi-stage optimisé
- [ ] Image de base sécurisée (Alpine/Distroless)
- [ ] Utilisateur non-root configuré
- [ ] Health check implémenté
- [ ] Taille d'image < 500MB

**Template Dockerfile Node.js :**
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:${PORT:-8080}/health || exit 1

USER nextjs
EXPOSE 8080
CMD ["node", "dist/app.js"]
```

### ✅ 3.2 Configuration Helm

**Critères :**
- [ ] Helm chart validé avec `helm lint`
- [ ] Values.yaml configuré par environnement
- [ ] Resource limits définis
- [ ] Health checks Kubernetes configurés
- [ ] Security context strict appliqué

**Values.yaml obligatoires :**
```yaml
deployment:
  replicaCount: 2
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi

healthCheck:
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10

securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  allowPrivilegeEscalation: false
```

### ✅ 3.3 Variables d'Environnement et Secrets

**Critères :**
- [ ] ConfigMaps créés pour configuration non-sensible
- [ ] Secrets Kubernetes créés pour données sensibles
- [ ] Variables d'environnement documentées
- [ ] Configuration par environnement (dev/staging/prod)

**Exemple ConfigMap :**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-config
data:
  SERVICE_NAME: "user-service"
  LOG_LEVEL: "info"
  API_GATEWAY_URL: "http://api-gateway-service:8080"
```

## 4. Critères de Monitoring et Observabilité

### ✅ 4.1 Health Checks Standardisés

**Endpoints obligatoires :**
- [ ] `GET /health` - État général du service
- [ ] `GET /ready` - Prêt à recevoir du trafic
- [ ] `GET /metrics` - Métriques Prometheus

**Implémentation health check :**
```typescript
// Exemple Node.js
app.get('/health', async (req, res) => {
  const health = {
    status: 'UP',
    timestamp: new Date().toISOString(),
    service: process.env.SERVICE_NAME,
    version: process.env.SERVICE_VERSION,
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      externalServices: await checkExternalServices()
    }
  };

  const isHealthy = Object.values(health.checks)
    .every(check => check.status === 'UP');

  res.status(isHealthy ? 200 : 503).json(health);
});
```

### ✅ 4.2 Logging Structuré

**Critères :**
- [ ] Format JSON structuré obligatoire
- [ ] Correlation IDs propagés
- [ ] Niveaux de log appropriés (debug/info/warn/error)
- [ ] Pas de données sensibles dans les logs
- [ ] Rotation des logs configurée

**Format de log obligatoire :**
```json
{
  "timestamp": "2024-01-19T10:30:00.000Z",
  "level": "info",
  "message": "HTTP Request",
  "service": "user-service",
  "version": "1.0.0",
  "correlation_id": "abc-123-def",
  "method": "POST",
  "url": "/api/v1/users",
  "status_code": 201,
  "response_time": 150
}
```

### ✅ 4.3 Métriques Prometheus

**Métriques obligatoires :**
- [ ] `http_request_duration_seconds` - Durée des requêtes HTTP
- [ ] `http_requests_total` - Nombre total de requêtes
- [ ] `http_request_errors_total` - Nombre d'erreurs HTTP
- [ ] `db_query_duration_seconds` - Durée des requêtes DB
- [ ] `business_operations_total` - Opérations métier

**Exemple métriques :**
```typescript
// Node.js avec prom-client
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});
```

## 5. Critères Spécifiques par Stack Technologique

### ✅ 5.1 Node.js + TypeScript

**Critères spécifiques :**
- [ ] NestJS configuré
- [ ] Prisma ORM avec schéma validé
- [ ] TypeScript strict mode activé
- [ ] Jest configuré avec coverage
- [ ] ESLint + Prettier configurés

**Configuration obligatoire :**
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true
  }
}

// package.json scripts
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/app.js",
    "dev": "ts-node-dev src/app.ts",
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```

### ✅ 5.2 Python + FastAPI

**Critères spécifiques :**
- [ ] FastAPI avec Pydantic models
- [ ] SQLAlchemy ORM configuré
- [ ] pytest avec coverage ≥ 80%
- [ ] Black + isort + mypy configurés
- [ ] Alembic pour migrations

**Configuration obligatoire :**
```python
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py311']

[tool.isort]
profile = "black"

[tool.mypy]
python_version = "3.11"
strict = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=app --cov-report=term-missing"
```

### ✅ 5.3 Go

**Critères spécifiques :**
- [ ] Gin ou Echo framework
- [ ] GORM configuré
- [ ] Go modules (go.mod) à jour
- [ ] Tests avec testify
- [ ] golangci-lint configuré

**Configuration obligatoire :**
```go
// go.mod
module github.com/visiobook/service-name

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    gorm.io/gorm v1.25.5
    github.com/stretchr/testify v1.8.4
)
```

### ✅ 5.4 Vue.js + TypeScript (Frontend)

**Critères spécifiques :**
- [ ] Vue 3 + Composition API
- [ ] Vite build tool configuré
- [ ] Vitest pour tests unitaires
- [ ] Cypress pour tests E2E
- [ ] ESLint + Prettier configurés

**Configuration obligatoire :**
```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "test:unit": "vitest",
    "test:e2e": "cypress run",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts"
  }
}
```

## 6. Critères API et Documentation

### ✅ 6.1 Convention de Nommage des Routes

**Critères :**
- [ ] Format standardisé : `/api/v{version}/{service-name}/{resource}`
- [ ] Ressources au pluriel
- [ ] Verbes HTTP appropriés (GET, POST, PUT, DELETE)
- [ ] Codes de statut HTTP corrects

**Exemples conformes :**
```
GET    /api/v1/users                 # Liste des utilisateurs
POST   /api/v1/users                 # Créer un utilisateur
GET    /api/v1/users/{id}            # Obtenir un utilisateur
PUT    /api/v1/users/{id}            # Mettre à jour un utilisateur
DELETE /api/v1/users/{id}            # Supprimer un utilisateur
GET    /api/v1/users/{id}/projects   # Projets d'un utilisateur
```

### ✅ 6.2 Documentation OpenAPI

**Critères :**
- [ ] Spécification OpenAPI 3.0+ complète
- [ ] Tous les endpoints documentés
- [ ] Schemas de requête/réponse définis
- [ ] Exemples fournis pour chaque endpoint
- [ ] Documentation accessible via `/api/docs`

**Exemple OpenAPI :**
```yaml
openapi: 3.0.0
info:
  title: User Service API
  version: 1.0.0
  description: Gestion des utilisateurs Visiobook

paths:
  /api/v1/users:
    post:
      summary: Créer un utilisateur
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: Utilisateur créé avec succès
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```

### ✅ 6.3 Gestion d'Erreurs Standardisée

**Critères :**
- [ ] Format d'erreur uniforme
- [ ] Codes de statut HTTP appropriés
- [ ] Messages d'erreur clairs et localisés
- [ ] Correlation ID inclus dans les erreurs
- [ ] Pas d'exposition d'informations sensibles

**Format d'erreur standardisé :**
```json
{
  "error": {
    "code": 400,
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      }
    ],
    "correlation_id": "abc-123-def",
    "timestamp": 1642598400000
  }
}
```

## 7. Critères de Performance et Scalabilité

### ✅ 7.1 Objectifs de Performance

**Critères par type de service :**

**Services Core (user, project, storage) :**
- [ ] Temps de réponse P95 < 500ms
- [ ] Throughput > 1000 req/s
- [ ] Disponibilité > 99.8%

**Services IA (ai-analysis, media-generation) :**
- [ ] Temps d'analyse < 30s par page
- [ ] Utilisation GPU 70-90%
- [ ] Disponibilité > 99.5%

**Frontend (web-user-portal) :**
- [ ] Temps de chargement < 2s
- [ ] Score Lighthouse > 90
- [ ] First Contentful Paint < 1.5s

### ✅ 7.2 Tests de Performance

**Critères :**
- [ ] Tests de charge configurés
- [ ] Benchmarks de performance établis
- [ ] Profiling mémoire effectué
- [ ] Tests de montée en charge validés

**Outils recommandés :**
```bash
# Tests de charge
k6 run performance-tests.js

# Profiling Node.js
node --inspect dist/app.js

# Profiling Python
py-spy top --pid <pid>

# Profiling Go
go tool pprof http://localhost:6060/debug/pprof/profile
```

## 8. Critères de Sécurité

### ✅ 8.1 Audit de Sécurité

**Critères obligatoires :**
- [ ] Scan de vulnérabilités passé (npm audit, safety, govulncheck)
- [ ] Analyse de code statique (SonarQube, CodeQL)
- [ ] Dépendances à jour et sécurisées
- [ ] Secrets non exposés dans le code

**Commandes de validation :**
```bash
# Node.js
npm audit --audit-level high
npm run security:check

# Python
safety check -r requirements.txt
bandit -r app/

# Go
govulncheck ./...
gosec ./...
```

### ✅ 8.2 Configuration Sécurisée

**Critères :**
- [ ] HTTPS obligatoire en production
- [ ] Headers de sécurité configurés
- [ ] Validation des entrées implémentée
- [ ] Rate limiting configuré
- [ ] CORS configuré restrictif

**Headers de sécurité obligatoires :**
```typescript
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

## 9. Critères de CI/CD

### ✅ 9.1 Pipeline CI/CD

**Étapes obligatoires :**
- [ ] Linting et formatage automatiques
- [ ] Tests unitaires et d'intégration
- [ ] Scan de sécurité automatique
- [ ] Build et push d'image Docker
- [ ] Déploiement automatique en staging
- [ ] Tests de fumée post-déploiement

**Configuration GitHub Actions :**
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:coverage
      - run: npm audit --audit-level high
```

### ✅ 9.2 Stratégie de Déploiement

**Critères :**
- [ ] Déploiement Blue-Green configuré
- [ ] Rollback automatique en cas d'échec
- [ ] Tests de fumée automatiques
- [ ] Monitoring post-déploiement

## 10. Checklist de Validation Finale

### ✅ 10.1 Checklist Pré-Développement

**Avant de commencer le développement :**
- [ ] Architecture du service définie et validée
- [ ] Dépendances identifiées et documentées
- [ ] Interfaces API spécifiées (OpenAPI)
- [ ] Modèles de données conçus
- [ ] Stratégie de tests planifiée
- [ ] Configuration d'environnement préparée

### ✅ 10.2 Checklist Pré-Déploiement

**Avant le déploiement en staging :**
- [ ] Tous les tests passent (unitaires, intégration, E2E)
- [ ] Couverture de code ≥ 80%
- [ ] Audit de sécurité passé
- [ ] Documentation à jour
- [ ] Health checks fonctionnels
- [ ] Métriques exposées
- [ ] Logs structurés configurés
- [ ] Helm chart validé
- [ ] Variables d'environnement configurées
- [ ] Secrets créés

### ✅ 10.3 Checklist Pré-Production

**Avant le déploiement en production :**
- [ ] Tests de charge validés
- [ ] Performance conforme aux SLA
- [ ] Monitoring et alerting configurés
- [ ] Runbook de support créé
- [ ] Plan de rollback testé
- [ ] Formation équipe support effectuée
- [ ] Validation métier obtenue

## 11. Critères Spécifiques par Type de Microservice

### ✅ 11.1 Services Core (user, project, storage)

**Critères additionnels :**
- [ ] Authentification JWT implémentée
- [ ] Autorisation basée sur les rôles
- [ ] Cache Redis configuré
- [ ] Connection pooling optimisé
- [ ] Backup automatique configuré

### ✅ 11.2 Services IA (ai-analysis, media-generation)

**Critères additionnels :**
- [ ] GPU correctement configuré et détecté
- [ ] Modèles IA chargés et validés
- [ ] Memory pooling GPU implémenté
- [ ] Batch processing configuré
- [ ] Monitoring GPU spécifique
- [ ] Fallback CPU en cas d'indisponibilité GPU

**Validation GPU :**
```python
# Vérification GPU
import torch
assert torch.cuda.is_available(), "GPU not available"
assert torch.cuda.device_count() > 0, "No GPU devices found"
print(f"GPU: {torch.cuda.get_device_name(0)}")
```

### ✅ 11.3 Services Support (monitoring, logging, config)

**Critères additionnels :**
- [ ] Haute disponibilité configurée (≥ 3 replicas)
- [ ] Persistence des données configurée
- [ ] Backup et restore automatiques
- [ ] Monitoring de second niveau (monitoring du monitoring)

### ✅ 11.4 Frontend (web-user-portal, admin-dashboard)

**Critères additionnels :**
- [ ] Progressive Web App (PWA) configurée
- [ ] Service Worker pour cache offline
- [ ] Optimisation des bundles (code splitting)
- [ ] Accessibilité WCAG 2.1 AA
- [ ] Tests E2E avec Cypress
- [ ] Performance budget défini

## 12. Métriques de Succès

### ✅ 12.1 Métriques Techniques

**Objectifs par service :**
```yaml
Services Core:
  - Response time P95: < 500ms
  - Throughput: > 1000 req/s
  - Error rate: < 0.1%
  - Uptime: > 99.8%

Services IA:
  - Analysis time: < 30s per page
  - GPU utilization: 70-90%
  - Model accuracy: > 85%
  - Uptime: > 99.5%

Frontend:
  - Load time: < 2s
  - Lighthouse score: > 90
  - Conversion rate: > 15%
  - User error rate: < 1%
```

### ✅ 12.2 Métriques Qualité

**Objectifs globaux :**
- [ ] Couverture de tests : > 80%
- [ ] Complexité cyclomatique : < 10
- [ ] Duplication de code : < 5%
- [ ] Vulnérabilités sécurité : 0
- [ ] Temps de build CI/CD : < 10min
- [ ] MTTR incidents : < 30min

## 13. Processus de Validation

### ✅ 13.1 Review Process

**Étapes de validation :**
1. **Auto-validation** : Développeur vérifie la checklist
2. **Peer review** : Code review par un pair
3. **Tech lead review** : Validation architecture et standards
4. **QA validation** : Tests fonctionnels et non-fonctionnels
5. **DevOps validation** : Infrastructure et déploiement
6. **Product validation** : Validation métier et UX

### ✅ 13.2 Critères de Blocage

**Blocages automatiques :**
- [ ] Tests en échec
- [ ] Couverture < 80%
- [ ] Vulnérabilités sécurité critiques
- [ ] Performance dégradée > 20%
- [ ] Health checks en échec

### ✅ 13.3 Processus d'Exception

**En cas de non-conformité :**
1. **Documentation** : Justification technique détaillée
2. **Approbation** : Validation tech lead + product owner
3. **Plan de remédiation** : Timeline de mise en conformité
4. **Suivi** : Review périodique jusqu'à résolution

## 14. Templates et Outils

### ✅ 14.1 Templates de Code

**Disponibles dans les repositories :**
- `visiobook-service-template-nodejs` : Template NestJS + TypeScript
- `visiobook-service-template-python` : Template FastAPI + SQLAlchemy
- `visiobook-service-template-go` : Template Gin + GORM
- `visiobook-frontend-template-vue` : Template Vue 3 + TypeScript

### ✅ 14.2 Outils de Validation

**Scripts automatisés :**
```bash
# Validation complète
./scripts/validate-service.sh

# Validation spécifique
./scripts/check-tests.sh
./scripts/check-security.sh
./scripts/check-performance.sh
./scripts/check-helm.sh
```

### ✅ 14.3 Documentation de Référence

**Liens utiles :**
- [Guidelines de développement](./visiobook-development-guidelines.md)
- [Architecture prioritaire](./visiobook-priority-architecture.md)
- [Templates Helm Charts](../infra-helm-charts/)
- [Pipelines CI/CD](./cicd-pipelines/)

## 15. Conclusion

Cette Definition of Ready garantit que chaque microservice Visiobook :

✅ **Respecte les standards** de qualité et de sécurité
✅ **S'intègre parfaitement** dans l'écosystème microservices
✅ **Peut être déployé** de manière sûre et automatisée
✅ **Est observable** et maintenable en production
✅ **Offre les performances** requises pour l'expérience utilisateur

### Prochaines Étapes

1. **Formation équipes** sur cette DoR
2. **Intégration dans les workflows** de développement
3. **Automatisation** des validations via CI/CD
4. **Amélioration continue** basée sur les retours d'expérience

---

**Document maintenu par** : Équipe Architecture Visiobook
**Version** : 1.0.0
**Dernière mise à jour** : 26 Août 2025
**Prochaine révision** : 26 Novembre 2025

---

## Annexes

### Annexe A - Checklist Rapide par Stack

#### Node.js + TypeScript
```bash
□ npm run lint
□ npm run type-check
□ npm run test:coverage (>80%)
□ npm audit --audit-level high
□ docker build . (success)
□ helm lint charts/
```

#### Python + FastAPI
```bash
□ black --check app/
□ mypy app/
□ pytest --cov=app --cov-fail-under=80
□ safety check -r requirements.txt
□ docker build . (success)
□ helm lint charts/
```

#### Go
```bash
□ golangci-lint run
□ go test -coverprofile=coverage.out ./...
□ govulncheck ./...
□ docker build . (success)
□ helm lint charts/
```

#### Vue.js + TypeScript
```bash
□ npm run lint
□ npm run type-check
□ npm run test:unit
□ npm run test:e2e
□ npm run build
□ docker build . (success)
```

### Annexe B - Contacts Support

- **Tech Lead** : `tech-lead@visiobook.com`
- **DevOps Team** : `devops@visiobook.com`
- **QA Team** : `qa@visiobook.com`
- **Support Slack** : `#visiobook-dev-support`

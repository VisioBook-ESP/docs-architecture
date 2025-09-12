# API Documentation - Core Database Service

## Vue d'ensemble du service

### Rôle et responsabilités
Le **Core Database Service** est le service fondamental qui gère toutes les connexions et opérations de base de données pour l'ensemble de l'écosystème Visiobook. Il centralise la gestion des pools de connexions, les transactions distribuées, et assure la cohérence des données.

### Justification de l'atomisation
- **Point unique de défaillance contrôlé** : Centralise la logique de base de données
- **Optimisation des performances** : Pool de connexions optimisé et cache intelligent
- **Sécurité** : Gestion centralisée des accès et chiffrement
- **Maintenance** : Migrations et sauvegardes centralisées

### Informations techniques
- **Port** : 8084
- **Technology Stack** : NestJS + TypeScript + Prisma
- **Databases** : PostgreSQL, Redis
- **Version API** : v1

## Architecture du service

```mermaid
graph TB
    subgraph "Core Database Service"
        API[API Layer<br/>NestJS + TypeScript]
        POOL[Connection Pool Manager<br/>Prisma + ioredis]
        CACHE[Cache Layer<br/>Redis Cluster]
        MIGRATE[Migration Manager<br/>Prisma Migrate]
        BACKUP[Backup Manager<br/>Automated Backups]
        GUARD[Auth Guard<br/>JWT Validation]
    end

    subgraph "Databases"
        PG[(PostgreSQL<br/>Primary Data)]
        REDIS[(Redis<br/>Cache + Sessions)]
    end

    API --> GUARD
    API --> POOL
    POOL --> PG
    POOL --> REDIS
    API --> CACHE
    API --> MIGRATE
    API --> BACKUP

    classDef service fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef database fill:#fff8e1,stroke:#f9a825,stroke-width:2px

    class API,POOL,CACHE,MIGRATE,BACKUP,GUARD service
    class PG,REDIS database
```

### Architecture de Données - Stratégie de Migration

#### Note sur l'approche architecturale actuelle

> **🏗️ Phase de développement décentralisé (Actuelle)**
>
> Pour faciliter le développement parallèle des microservices, chaque service gère actuellement ses propres tables pour les objets dont il est responsable. Les objets provenant d'autres services sont "mockés" localement dans chaque base de données.
>
> **🎯 Phase de centralisation future (Roadmap)**
>
> Le Core Database Service récupérera progressivement les fichiers de migration de tous les microservices et centralisera l'accès aux données via des contrats d'interface standardisés.

#### Schémas de Référence Centralisés (Future)

Les schémas ci-dessous représentent la structure de données unifiée qui sera mise en place lors de la centralisation :

```sql
-- Unified Users table (source: Core User Service)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role VARCHAR(50) DEFAULT 'user',
    subscription_type VARCHAR(50) DEFAULT 'free',
    email_verified BOOLEAN DEFAULT FALSE,
    phone_verified BOOLEAN DEFAULT FALSE,
    mfa_enabled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Unified Projects table (source: Core Project Service)
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'draft',
    visibility VARCHAR(50) DEFAULT 'private',
    source_content_id UUID,
    generated_content_id UUID,
    settings JSONB DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Unified Files table (source: Support Storage Service)
CREATE TABLE files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    project_id UUID REFERENCES projects(id),
    original_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_type VARCHAR(100) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    size_bytes BIGINT NOT NULL,
    checksum_md5 VARCHAR(32) NOT NULL,
    storage_provider VARCHAR(50) DEFAULT 'azure_blob',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Unified Analysis Jobs table (source: AI Analysis Service)
CREATE TABLE analysis_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    job_type VARCHAR(100) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    parameters JSONB NOT NULL,
    result JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP
);
```

#### Stratégie de Migration des Microservices

##### Phase 1 - Collecte des Migrations (En cours)
1. **Récupération automatique** des fichiers de migration de chaque service
2. **Analyse des schémas** pour identifier les dépendances et conflits
3. **Mapping des objets** entre services (utilisateurs, projets, fichiers, etc.)
4. **Validation des contrats** d'interface proposés

##### Phase 2 - Implémentation des Contrats (Future)
1. **Déploiement des contrats** d'interface standardisés
2. **Migration progressive** service par service
3. **Tests de compatibilité** et validation des données
4. **Monitoring** des performances et de la fiabilité

##### Phase 3 - Centralisation Complète (Future)
1. **Décommissionnement** des bases de données locales
2. **Optimisation** des requêtes centralisées
3. **Mise en place** de la haute disponibilité
4. **Documentation** finale et formation des équipes

#### Processus de Récupération des Migrations

```bash
# Script de collecte automatique des migrations
#!/bin/bash

# Collecte depuis chaque microservice
collect_migrations() {
    services=("core-user-service" "core-project-service" "support-storage-service" "ai-analysis-service")

    for service in "${services[@]}"; do
        echo "Collecting migrations from $service..."

        # Récupération des fichiers de migration
        rsync -av $service/migrations/ ./collected-migrations/$service/

        # Analyse des dépendances
        python analyze_dependencies.py ./collected-migrations/$service/

        # Génération des contrats d'interface
        python generate_contracts.py $service
    done
}

# Validation des contrats
validate_contracts() {
    echo "Validating interface contracts..."
    python validate_contracts.py ./contracts/
}

# Génération du schéma unifié
generate_unified_schema() {
    echo "Generating unified database schema..."
    python merge_schemas.py ./collected-migrations/ > unified_schema.sql
}
```

### Variables d'environnement

```bash
# Database connections
DATABASE_URL=postgresql://user:password@localhost:5432/visiobook
REDIS_URL=redis://localhost:6379

# Connection pools
DB_POOL_MIN=5
DB_POOL_MAX=20
DB_POOL_IDLE_TIMEOUT=30000
REDIS_POOL_SIZE=10

# NestJS Configuration
NODE_ENV=production
PORT=8084

# Security
DB_ENCRYPTION_KEY=your-encryption-key
JWT_SECRET=your-jwt-secret

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=info

# Prisma
PRISMA_SCHEMA_DISABLE_ADVISORY_LOCK=1
```

## Authentification et sécurité

> **📋 Référence** : Voir [Règles Communes](./regles_communes.md) pour les standards d'authentification, permissions et sécurité.

### Système JWT
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json"
}
```

### Structure du JWT Token
```json
{
  "sub": "user_uuid",
  "email": "user@example.com",
  "role": "user|premium|admin",
  "subscription_type": "free|premium",
  "permissions": ["domain:action:resource"],
  "iat": 1642234567,
  "exp": 1642320967,
  "jti": "token_unique_id"
}
```

### Niveaux de permissions
- **admin** : Accès complet aux opérations de maintenance et monitoring
- **user** : Accès aux opérations CRUD sur ses propres données
- **premium** : Accès étendu avec quotas augmentés

### Headers de sécurité requis
```http
Authorization: Bearer <jwt_token>
X-Request-ID: <unique_request_id>
X-Client-Version: <client_version>
```

## Endpoints API

### Health & Monitoring

#### GET /health
**Description** : Vérification de l'état du service et des connexions DB

**Permissions** : Aucune

**Réponse** :
```json
{
  "status": "UP",
  "timestamp": "2024-01-15T10:30:00Z",
  "service": "core-database-service",
  "version": "1.0.0",
  "checks": {
    "postgresql": {
      "status": "UP",
      "responseTime": "15ms",
      "connections": {
        "active": 8,
        "idle": 12,
        "total": 20
      }
    },
    "redis": {
      "status": "UP",
      "responseTime": "2ms",
      "memory": "45MB"
    },
    "prisma": {
      "status": "UP",
      "responseTime": "8ms",
      "migrations": "up_to_date"
    }
  }
}
```

#### GET /metrics
**Description** : Métriques Prometheus

**Permissions** : admin

### Connection Management

#### POST /api/v1/connections/acquire
**Description** : Acquisition d'une connexion DB pour un service

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "service_name": "core-user-service",
  "database_type": "postgresql",
  "operation_type": "read" // ou "write"
}
```

**Réponse** :
```json
{
  "connection_id": "conn_123456789",
  "database_url": "postgresql://...",
  "expires_at": "2024-01-15T10:35:00Z",
  "pool_stats": {
    "active": 9,
    "idle": 11,
    "waiting": 0
  }
}
```

#### POST /api/v1/connections/release
**Description** : Libération d'une connexion DB

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "connection_id": "conn_123456789",
  "service_name": "core-user-service"
}
```

**Réponse** :
```json
{
  "status": "released",
  "connection_id": "conn_123456789",
  "duration_ms": 1250
}
```

### Data Operations

#### POST /api/v1/data/query
**Description** : Exécution de requêtes sécurisées

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "database": "postgresql",
  "query_type": "select",
  "table": "projects",
  "filters": {
    "user_id": "uuid",
    "status": "active"
  },
  "pagination": {
    "page": 1,
    "limit": 20
  }
}
```

**Réponse** :
```json
{
  "data": [
    {
      "id": "proj_123",
      "title": "Mon projet",
      "status": "active",
      "created_at": "2024-01-15T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "pages": 3
  },
  "execution_time_ms": 25
}
```

#### POST /api/v1/data/transaction
**Description** : Exécution de transactions distribuées

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "operations": [
    {
      "database": "postgresql",
      "operation": "insert",
      "table": "projects",
      "data": {
        "title": "Nouveau projet",
        "user_id": "user_123"
      }
    },
    {
      "database": "redis",
      "operation": "set",
      "key": "project:cache:proj_new",
      "value": "{\"status\": \"created\"}",
      "ttl": 3600
    }
  ]
}
```

**Réponse** :
```json
{
  "transaction_id": "tx_789456123",
  "status": "committed",
  "results": [
    {
      "operation": 1,
      "status": "success",
      "data": {
        "id": "proj_new_123",
        "created_at": "2024-01-15T10:30:00Z"
      }
    },
    {
      "operation": 2,
      "status": "success"
    }
  ],
  "execution_time_ms": 45
}
```

### Migration Management

#### GET /api/v1/migrations/status
**Description** : État des migrations de base de données

**Permissions** : admin

**Réponse** :
```json
{
  "current_version": "20240115_001",
  "pending_migrations": [
    {
      "version": "20240116_001",
      "description": "Add indexes for performance",
      "estimated_duration": "2 minutes"
    }
  ],
  "last_migration": {
    "version": "20240115_001",
    "applied_at": "2024-01-15T09:00:00Z",
    "duration_ms": 1500
  }
}
```

#### POST /api/v1/migrations/run
**Description** : Exécution des migrations en attente

**Permissions** : admin

**Requête** :
```json
{
  "target_version": "20240116_001", // optionnel
  "dry_run": false
}
```

**Réponse** :
```json
{
  "migration_id": "mig_123456",
  "status": "running",
  "progress": {
    "current_step": 1,
    "total_steps": 3,
    "estimated_remaining": "1 minute"
  }
}
```

## Flux de transactions CRUD

### Diagramme de séquence - Opération de lecture

```mermaid
sequenceDiagram
    participant Client as Service Client
    participant API as Database API
    participant Pool as Connection Pool
    participant Cache as Redis Cache
    participant DB as PostgreSQL

    Client->>API: POST /api/v1/data/query
    API->>Cache: Check cache for query
    alt Cache Hit
        Cache-->>API: Return cached data
        API-->>Client: Return data (cached)
    else Cache Miss
        API->>Pool: Acquire connection
        Pool->>DB: Get connection
        DB-->>Pool: Connection ready
        Pool-->>API: Connection acquired
        API->>DB: Execute query
        DB-->>API: Query results
        API->>Cache: Store in cache (TTL: 300s)
        API->>Pool: Release connection
        API-->>Client: Return data
    end
```

### Diagramme de flux - Transaction distribuée

```mermaid
flowchart TD
    Start([Transaction Request]) --> Validate{Validate Operations}
    Validate -->|Invalid| Error[Return Error 400]
    Validate -->|Valid| Begin[Begin Transaction]

    Begin --> AcquireConn[Acquire Connections]
    AcquireConn --> CheckConn{All Connections OK?}
    CheckConn -->|No| ReleaseConn[Release Connections]
    ReleaseConn --> Error503[Return Error 503]

    CheckConn -->|Yes| ExecuteOps[Execute Operations]
    ExecuteOps --> CheckResults{All Operations Success?}

    CheckResults -->|No| Rollback[Rollback Transaction]
    Rollback --> ReleaseConn

    CheckResults -->|Yes| Commit[Commit Transaction]
    Commit --> UpdateCache[Update Cache]
    UpdateCache --> ReleaseConn2[Release Connections]
    ReleaseConn2 --> Success[Return Success 200]

    Error --> End([End])
    Error503 --> End
    Success --> End
```

## Parcours utilisateurs détaillés

### Milestone 1: Importer un contenu

**US 1.1 - Import de fichiers**
```
1. POST /api/v1/data/transaction
   - Insert project record
   - Store file metadata
   - Update user project count

2. POST /api/v1/connections/acquire (for ai-analysis-service)
   - Acquire connection for content analysis
```

**US 1.5 - Extraction des scènes clés**
```
1. POST /api/v1/data/query
   - Retrieve project content
   - Get analysis parameters

2. POST /api/v1/data/transaction
   - Store scene analysis results
   - Update project status
```

### Milestone 3: Générer et visualiser une animation

**US 3.1 - Génération automatique**
```
1. POST /api/v1/data/query
   - Get project data and analysis
   - Retrieve user preferences

2. POST /api/v1/data/transaction
   - Create generation job record
   - Update project status to "generating"
```

### Milestone 5: Historique et réutilisation

**US 5.1 - Accéder à l'historique**
```
1. POST /api/v1/data/query
   - Query user projects with pagination
   - Include project metadata and status
```

**US 5.2 - Modifier un projet existant**
```
1. POST /api/v1/data/transaction
   - Create project version backup
   - Update project with new parameters
   - Invalidate related cache entries
```

## Codes d'erreur

| Code | Message | Description |
|------|---------|-------------|
| 200 | Success | Opération réussie |
| 400 | Bad Request | Requête malformée ou paramètres invalides |
| 401 | Unauthorized | Token JWT manquant ou invalide |
| 403 | Forbidden | Permissions insuffisantes |
| 404 | Not Found | Ressource non trouvée |
| 409 | Conflict | Conflit de données (ex: email déjà utilisé) |
| 429 | Too Many Requests | Limite de taux dépassée |
| 500 | Internal Server Error | Erreur interne du serveur |
| 503 | Service Unavailable | Base de données indisponible |
| 504 | Gateway Timeout | Timeout de connexion à la base de données |

### Format d'erreur standardisé
```json
{
  "error": {
    "code": "VISIOBOOK_SERVICE_UNAVAILABLE",
    "message": "Unable to acquire database connection",
    "details": {
      "database": "postgresql",
      "pool_status": "exhausted",
      "retry_after": 30
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_123456789",
    "service": "core-database-service"
  }
}
```

## Contrats d'Interface

### Vue d'ensemble
Le Core Database Service prépare la centralisation progressive des données des microservices. Les contrats d'interface suivants définissent les APIs futures qui permettront aux autres services de migrer vers une architecture centralisée.

### Interface User Contract
```typescript
interface UserContract {
  // Données de base
  getUserById(id: string): Promise<User>;
  getUserByEmail(email: string): Promise<User>;
  createUser(userData: CreateUserData): Promise<User>;
  updateUser(id: string, userData: UpdateUserData): Promise<User>;
  deleteUser(id: string): Promise<boolean>;

  // Authentification
  validateCredentials(email: string, password: string): Promise<AuthResult>;
  createSession(userId: string, deviceInfo: DeviceInfo): Promise<Session>;
  validateSession(sessionToken: string): Promise<Session>;
  refreshSession(refreshToken: string): Promise<Session>;

  // Permissions et rôles
  getUserPermissions(userId: string): Promise<Permission[]>;
  hasPermission(userId: string, permission: string): Promise<boolean>;
  updateUserRole(userId: string, role: string): Promise<boolean>;

  // Profils utilisateur
  getUserProfile(userId: string): Promise<UserProfile>;
  updateUserProfile(userId: string, profileData: UpdateProfileData): Promise<UserProfile>;
}

interface User {
  id: string;
  email: string;
  username?: string;
  first_name?: string;
  last_name?: string;
  role: 'user' | 'premium' | 'admin';
  subscription_type: 'free' | 'premium';
  email_verified: boolean;
  phone_verified: boolean;
  mfa_enabled: boolean;
  created_at: string;
  updated_at: string;
}
```

### Interface Project Contract
```typescript
interface ProjectContract {
  // CRUD de base
  getProjectById(id: string): Promise<Project>;
  getProjectsByUser(userId: string, filters?: ProjectFilters): Promise<Project[]>;
  createProject(projectData: CreateProjectData): Promise<Project>;
  updateProject(id: string, projectData: UpdateProjectData): Promise<Project>;
  deleteProject(id: string): Promise<boolean>;

  // Versioning
  getProjectVersions(projectId: string): Promise<ProjectVersion[]>;
  createProjectVersion(projectId: string, versionData: VersionData): Promise<ProjectVersion>;
  restoreProjectVersion(projectId: string, versionNumber: number): Promise<boolean>;

  // Workflows
  getProjectWorkflows(projectId: string): Promise<Workflow[]>;
  createWorkflow(projectId: string, workflowData: WorkflowData): Promise<Workflow>;
  updateWorkflowStatus(workflowId: string, status: string): Promise<boolean>;
}

interface Project {
  id: string;
  user_id: string;
  title: string;
  description?: string;
  status: 'draft' | 'active' | 'completed' | 'archived';
  visibility: 'private' | 'shared' | 'public';
  settings: ProjectSettings;
  metadata: ProjectMetadata;
  created_at: string;
  updated_at: string;
}
```

### Interface Storage Contract
```typescript
interface StorageContract {
  // Gestion des fichiers
  getFileById(id: string): Promise<FileMetadata>;
  getFilesByProject(projectId: string): Promise<FileMetadata[]>;
  getFilesByUser(userId: string, filters?: FileFilters): Promise<FileMetadata[]>;
  createFileRecord(fileData: CreateFileData): Promise<FileMetadata>;
  updateFileMetadata(id: string, metadata: UpdateFileData): Promise<FileMetadata>;
  deleteFile(id: string): Promise<boolean>;

  // Transformations
  getFileTransformations(fileId: string): Promise<Transformation[]>;
  createTransformation(transformData: CreateTransformData): Promise<Transformation>;
  getTransformationStatus(transformId: string): Promise<TransformationStatus>;
  updateTransformationProgress(transformId: string, progress: number): Promise<boolean>;

  // CDN et cache
  getCDNUrls(fileId: string): Promise<CDNUrls>;
  invalidateCDNCache(fileIds: string[]): Promise<boolean>;
  getStorageStats(userId: string): Promise<StorageStats>;
}

interface FileMetadata {
  id: string;
  user_id: string;
  project_id?: string;
  original_name: string;
  file_path: string;
  file_type: string;
  mime_type: string;
  size_bytes: number;
  checksum_md5: string;
  storage_provider: string;
  is_public: boolean;
  metadata: Record<string, any>;
  created_at: string;
  updated_at: string;
}
```

### Interface AI Contract
```typescript
interface AIContract {
  // Analyses
  getAnalysisJobs(projectId: string): Promise<AnalysisJob[]>;
  createAnalysisJob(jobData: CreateAnalysisData): Promise<AnalysisJob>;
  getJobStatus(jobId: string): Promise<JobStatus>;
  updateJobProgress(jobId: string, progress: number): Promise<boolean>;
  getAnalysisResults(jobId: string): Promise<AnalysisResult>;

  // Générations
  getGenerationJobs(projectId: string): Promise<GenerationJob[]>;
  createGenerationJob(jobData: CreateGenerationData): Promise<GenerationJob>;
  getGenerationStatus(jobId: string): Promise<GenerationStatus>;
  getGenerationResults(jobId: string): Promise<GenerationResult>;

  // Modèles et performance
  getAvailableModels(): Promise<AIModel[]>;
  getModelPerformance(modelName: string): Promise<ModelPerformance>;
  updateModelMetrics(modelName: string, metrics: ModelMetrics): Promise<boolean>;
}

interface AnalysisJob {
  id: string;
  project_id: string;
  job_type: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  progress_percent: number;
  parameters: Record<string, any>;
  result?: AnalysisResult;
  error_message?: string;
  created_at: string;
  started_at?: string;
  completed_at?: string;
}
```

### Stratégie de Migration des Contrats

#### Phase 1 - Implémentation des contrats (En cours)
- Développement des interfaces TypeScript
- Implémentation des endpoints correspondants
- Tests avec données mockées
- Documentation des contrats

#### Phase 2 - Migration progressive (Future)
- Migration service par service vers les contrats
- Maintien de la compatibilité avec les APIs existantes
- Monitoring des performances et de la fiabilité
- Formation des équipes de développement

#### Phase 3 - Consolidation (Future)
- Décommissionnement des connexions directes
- Optimisation des performances centralisées
- Mise en place de la haute disponibilité
- Documentation finale et formation

### Endpoints de Contrats (Futurs)

#### GET /api/v1/contracts/user/{user_id}
**Description** : Accès aux données utilisateur via contrat centralisé

**Permissions** : user, premium, admin

**Réponse** :
```json
{
  "user": {
    "id": "user_123456789",
    "email": "user@example.com",
    "role": "premium",
    "subscription_type": "premium",
    "profile": {
      "first_name": "John",
      "last_name": "Doe",
      "avatar_url": "https://cdn.visiobook.com/avatars/user_123.jpg"
    }
  },
  "source": "centralized_contract",
  "cache_status": "hit",
  "response_time_ms": 15
}
```

#### GET /api/v1/contracts/project/{project_id}
**Description** : Accès aux données projet via contrat centralisé

**Permissions** : user, premium, admin

**Réponse** :
```json
{
  "project": {
    "id": "proj_123456789",
    "title": "Mon Projet",
    "status": "active",
    "workflows": [
      {
        "id": "workflow_456789",
        "type": "ai_analysis",
        "status": "completed"
      }
    ]
  },
  "source": "centralized_contract",
  "aggregated_from": ["project_service", "user_service", "ai_service"]
}
```

## Versioning et migration

### Convention v1
- **URL Base** : `/api/v1/`
- **Headers** : `Accept: application/vnd.visiobook.v1+json`
- **Rétrocompatibilité** : Maintenue pendant 12 mois minimum

### Stratégie de migration
1. **Dépréciation** : Annonce 6 mois avant suppression
2. **Coexistence** : v1 et v2 fonctionnent en parallèle
3. **Migration automatique** : Scripts de migration des données
4. **Documentation** : Guide de migration détaillé

### Exemple de migration v1 → v2
```json
{
  "migration": {
    "from_version": "v1",
    "to_version": "v2",
    "changes": [
      {
        "type": "endpoint_renamed",
        "old": "/api/v1/data/query",
        "new": "/api/v2/data/search"
      },
      {
        "type": "field_added",
        "endpoint": "/api/v2/data/search",
        "field": "search_options"
      }
    ],
    "breaking_changes": false,
    "migration_deadline": "2024-12-31"
  }
}

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
- **Port** : 8095
- **Technology Stack** : Node.js 18 + TypeScript + Prisma + TypeORM
- **Databases** : PostgreSQL, Redis, CosmosDB, Vector DB
- **Version API** : v1

## Architecture du service

```mermaid
graph TB
    subgraph "Core Database Service"
        API[API Layer<br/>Express + TypeScript]
        POOL[Connection Pool Manager<br/>pg-pool + ioredis]
        CACHE[Cache Layer<br/>Redis Cluster]
        MIGRATE[Migration Manager<br/>Prisma + TypeORM]
        BACKUP[Backup Manager<br/>Automated Backups]
    end

    subgraph "Databases"
        PG[(PostgreSQL<br/>Primary Data)]
        REDIS[(Redis<br/>Cache + Sessions)]
        COSMOS[(CosmosDB<br/>NoSQL Data)]
        VECTOR[(Vector DB<br/>AI Embeddings)]
    end

    API --> POOL
    POOL --> PG
    POOL --> REDIS
    POOL --> COSMOS
    POOL --> VECTOR
    API --> CACHE
    API --> MIGRATE
    API --> BACKUP

    classDef service fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef database fill:#fff8e1,stroke:#f9a825,stroke-width:2px

    class API,POOL,CACHE,MIGRATE,BACKUP service
    class PG,REDIS,COSMOS,VECTOR database
```

### Schémas de base de données

#### PostgreSQL - Tables principales
```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) DEFAULT 'user',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);

-- Projects table
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'draft',
    content_path VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_projects_user_id ON projects(user_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_created_at ON projects(created_at);

-- Content analysis table
CREATE TABLE content_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    analysis_type VARCHAR(100) NOT NULL,
    result JSONB NOT NULL,
    confidence_score DECIMAL(3,2),
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_content_analysis_project_id ON content_analysis(project_id);
CREATE INDEX idx_content_analysis_type ON content_analysis(analysis_type);
CREATE INDEX idx_content_analysis_confidence ON content_analysis(confidence_score);

-- Media assets table
CREATE TABLE media_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    asset_type VARCHAR(50) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    metadata JSONB,
    size_bytes BIGINT,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_media_assets_project_id ON media_assets(project_id);
CREATE INDEX idx_media_assets_type ON media_assets(asset_type);
```

### Variables d'environnement

```bash
# Database connections
DATABASE_URL=postgresql://user:password@localhost:5432/visiobook
REDIS_URL=redis://localhost:6379
COSMOS_CONNECTION_STRING=AccountEndpoint=https://...
VECTOR_DB_URL=https://your-vector-db.com

# Connection pools
DB_POOL_MIN=5
DB_POOL_MAX=20
DB_POOL_IDLE_TIMEOUT=30000
REDIS_POOL_SIZE=10

# Security
DB_ENCRYPTION_KEY=your-encryption-key
JWT_SECRET=your-jwt-secret

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=info
```

## Authentification et sécurité

### Système JWT
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json"
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
    "cosmosdb": {
      "status": "UP",
      "responseTime": "25ms"
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
    "code": "DATABASE_CONNECTION_FAILED",
    "message": "Unable to acquire database connection",
    "details": {
      "database": "postgresql",
      "pool_status": "exhausted",
      "retry_after": 30
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_123456789"
  }
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

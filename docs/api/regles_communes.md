# Règles Communes - API Visiobook

## Vue d'ensemble

Ce document définit les standards et conventions communes à tous les microservices de l'écosystème Visiobook. Tous les services doivent se conformer à ces règles pour assurer la cohérence et l'interopérabilité.

## 🌐 URLs et Endpoints

### Convention d'URLs de Production
```
https://api.visiobook.com/core-user-service              # Core User Service
https://api.visiobook.com/core-project-service           # Core Project Service
https://api.visiobook.com/support-storage-service        # Support Storage Service
https://api.visiobook.com/ai-analysis-service            # AI Analysis Service
https://api.visiobook.com/core-database-service          # Core Database Service
```

### Convention d'URLs de Développement
```
http://localhost:8081   # Core User Service
http://localhost:8086   # Core Project Service
http://localhost:8089   # Support Storage Service
http://localhost:8083   # AI Analysis Service
http://localhost:8084   # Core Database Service
http://localhost:3000   # Web User Portal
```

### Structure des Endpoints
- **Base** : `/api/v1/`
- **Format** : `/api/v1/{resource}/{action}`
- **Exemples** :
  - `GET /api/v1/users/profile`
  - `POST /api/v1/projects`
  - `PUT /api/v1/storage/files/{id}`

## 🔐 Authentification JWT

### Structure Standard du Token
```json
{
  "sub": "user_uuid",
  "email": "user@example.com",
  "role": "user|premium|admin",
  "subscription_type": "free|premium",
  "iat": 1642234567,
  "exp": 1642320967,
  "jti": "token_unique_id"
}
```

### Gestion des Permissions
Les permissions dans Visiobook sont basées sur un système de rôles simples, sans granularité fine :

- **Pas de champ `permissions` dans le JWT** : Les permissions sont dérivées directement du champ `role`
- **Contrôle d'accès simplifié** : Chaque endpoint vérifie le rôle requis (user/premium/admin)
- **Héritage des permissions** : admin > premium > user (les rôles supérieurs incluent les permissions inférieures)

```json
{
  "permission_model": "role_based_simple",
  "granularity": "service_level",
  "inheritance": true,
  "roles_hierarchy": ["user", "premium", "admin"]
}
```

### Durées de Validité
- **Access Token** : 15 minutes (900 secondes)
- **Refresh Token** : 7 jours (604800 secondes)
- **Session** : 24 heures (86400 secondes)

### Headers d'Authentification
```http
Authorization: Bearer <jwt_access_token>
X-Refresh-Token: <jwt_refresh_token>
```

## 🛡️ Système de Rôles

### Rôles Standards
- **user** : Accès de base aux fonctionnalités standard
  - Gestion de son profil utilisateur
  - Création et gestion de ses propres projets
  - Upload et gestion de ses fichiers personnels
  - Utilisation des fonctionnalités IA de base (analyse de contenu, génération d'images)
  - Quotas standards (projets limités, stockage de base)

- **premium** : Accès étendu avec fonctionnalités avancées et quotas augmentés
  - Toutes les fonctionnalités du rôle `user`
  - Fonctionnalités IA avancées (génération audio, génération vidéo)
  - Quotas étendus (plus de projets, stockage augmenté)
  - Priorité dans les files de traitement IA
  - Accès aux modèles IA haute qualité

- **admin** : Accès complet à tous les services et données
  - Toutes les fonctionnalités des rôles `user` et `premium`
  - Gestion de tous les utilisateurs et leurs données
  - Accès à tous les projets de la plateforme
  - Gestion des fichiers de tous les utilisateurs
  - Administration des services et monitoring
  - Gestion des modèles IA et des ressources système

### Contrôle d'Accès par Rôle
```json
{
  "user": {
    "own_data_only": true,
    "ai_features": ["content_analysis", "image_generation"],
    "quotas": {
      "projects": 10,
      "storage_gb": 5,
      "ai_requests_per_day": 100
    }
  },
  "premium": {
    "own_data_only": true,
    "ai_features": ["content_analysis", "image_generation", "audio_generation", "video_generation"],
    "quotas": {
      "projects": 50,
      "storage_gb": 50,
      "ai_requests_per_day": 1000
    },
    "priority_processing": true
  },
  "admin": {
    "own_data_only": false,
    "access_all_data": true,
    "ai_features": ["all"],
    "quotas": {
      "projects": "unlimited",
      "storage_gb": "unlimited",
      "ai_requests_per_day": "unlimited"
    },
    "system_management": true
  }
}
```

## 📋 Headers de Sécurité

### Headers Obligatoires
```http
Authorization: Bearer <jwt_token>
Content-Type: application/json
X-Request-ID: <unique_request_id>
X-Client-Version: <client_version>
```

### Headers Optionnels par Contexte
```http
# Upload de fichiers (Support Storage Service)
X-File-Checksum: <md5_checksum>
X-Upload-Session: <upload_session_id>
X-Chunk-Index: <chunk_number>
X-Chunk-Checksum: <chunk_md5>

# Traitement IA (AI Analysis Service)
X-GPU-Priority: low|normal|high
X-Content-Hash: <sha256_hash>
X-Model-Version: <model_version>

# Gestion de projets (Core Project Service)
X-Project-Version: <version_number>
X-Workflow-Priority: low|normal|high

# Cache et performance
X-Cache-Control: no-cache|max-age=3600
X-Preferred-Region: eu-west|us-east

# Monitoring et debugging
X-Debug-Mode: true|false
X-Trace-ID: <distributed_trace_id>
```

### Headers de Réponse Standards
```http
X-Request-ID: <same_as_request>
X-Response-Time: <processing_time_ms>
X-Rate-Limit-Remaining: <remaining_requests>
X-Rate-Limit-Reset: <reset_timestamp>
Cache-Control: public, max-age=3600
```

## 📄 Formats de Réponse Standards

### Structure de Pagination
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "pages": 8,
    "has_next": true,
    "has_previous": false
  }
}
```

### Structure de Métadonnées
```json
{
  "data": {...},
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "version": "v1",
    "source": "service_name",
    "cache_status": "hit|miss|stale",
    "processing_time_ms": 150
  }
}
```

### Structure de Réponse avec Statut
```json
{
  "status": "success|pending|failed",
  "data": {...},
  "message": "Human readable status message",
  "progress": {
    "current": 65,
    "total": 100,
    "estimated_completion": "2024-01-15T10:45:00Z"
  }
}
```

### Structure de Réponse de Liste avec Filtres
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "pages": 8
  },
  "filters_applied": {
    "status": ["active", "pending"],
    "date_range": {
      "from": "2024-01-01",
      "to": "2024-01-31"
    }
  },
  "summary": {
    "total_items": 156,
    "filtered_items": 45,
    "categories": {
      "active": 30,
      "pending": 15
    }
  }
}
```

## ⚠️ Codes d'Erreur

### Format Standard
```json
{
  "error": {
    "code": "VISIOBOOK_ERROR_CODE",
    "message": "Human readable error message",
    "details": {
      "field": "specific_field",
      "value": "invalid_value",
      "constraint": "validation_rule"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_123456789",
    "service": "service_name"
  }
}
```

### Codes d'Erreur Standards
```json
{
  "VISIOBOOK_INVALID_CREDENTIALS": "Invalid email or password",
  "VISIOBOOK_TOKEN_EXPIRED": "JWT token has expired",
  "VISIOBOOK_TOKEN_INVALID": "JWT token is invalid or malformed",
  "VISIOBOOK_INSUFFICIENT_PERMISSIONS": "User lacks required permissions",
  "VISIOBOOK_QUOTA_EXCEEDED": "User quota exceeded for this operation",
  "VISIOBOOK_RESOURCE_NOT_FOUND": "Requested resource does not exist",
  "VISIOBOOK_RESOURCE_CONFLICT": "Resource already exists or conflicts",
  "VISIOBOOK_VALIDATION_FAILED": "Input validation failed",
  "VISIOBOOK_FILE_TOO_LARGE": "File size exceeds maximum allowed limit",
  "VISIOBOOK_FILE_TYPE_INVALID": "File type not supported",
  "VISIOBOOK_UPLOAD_FAILED": "File upload operation failed",
  "VISIOBOOK_TRANSFORMATION_FAILED": "File transformation failed",
  "VISIOBOOK_AI_PROCESSING_FAILED": "AI processing operation failed",
  "VISIOBOOK_GPU_UNAVAILABLE": "GPU resources temporarily unavailable",
  "VISIOBOOK_SERVICE_UNAVAILABLE": "Backend service temporarily unavailable",
  "VISIOBOOK_RATE_LIMIT_EXCEEDED": "Rate limit exceeded for this endpoint",
  "VISIOBOOK_MAINTENANCE_MODE": "Service in maintenance mode",
  "VISIOBOOK_INTERNAL_ERROR": "Internal server error occurred",
  "VISIOBOOK_NETWORK_ERROR": "Unable to connect to backend services",
  "VISIOBOOK_TOKEN_REFRESH_FAILED": "Session expired, please log in again",
  "VISIOBOOK_UPLOAD_INTERRUPTED": "File upload was interrupted",
  "VISIOBOOK_CACHE_MISS": "Data not available offline",
  "VISIOBOOK_SERVICE_DEGRADED": "Some features may be limited due to service issues",
  "VISIOBOOK_QUOTA_WARNING": "Approaching usage limits",
  "VISIOBOOK_BROWSER_UNSUPPORTED": "Browser version not supported",
  "VISIOBOOK_GPU_MEMORY_EXHAUSTED": "Insufficient GPU memory for processing",
  "VISIOBOOK_UPLOAD_SESSION_EXPIRED": "Upload session has expired, please restart",
  "VISIOBOOK_WORKFLOW_TIMEOUT": "Workflow processing exceeded maximum time limit",
  "VISIOBOOK_MODEL_LOADING_FAILED": "AI model failed to load properly",
  "VISIOBOOK_CONTENT_TOO_COMPLEX": "Content is too complex for automatic processing",
  "VISIOBOOK_GENERATION_QUEUE_FULL": "Generation queue is full, please try again later",
  "VISIOBOOK_INSUFFICIENT_STORAGE": "Insufficient storage space available",
  "VISIOBOOK_CONCURRENT_MODIFICATION": "Resource is being modified by another process",
  "VISIOBOOK_BACKUP_FAILED": "Backup operation failed",
  "VISIOBOOK_MIGRATION_IN_PROGRESS": "Service migration in progress, limited functionality"
}
```

### Codes HTTP Standards
| Code | Usage | Description |
|------|-------|-------------|
| 200 | Success | Opération réussie |
| 201 | Created | Ressource créée avec succès |
| 202 | Accepted | Requête acceptée, traitement en cours |
| 204 | No Content | Opération réussie sans contenu de retour |
| 400 | Bad Request | Requête malformée ou données invalides |
| 401 | Unauthorized | Authentification requise ou échouée |
| 403 | Forbidden | Permissions insuffisantes |
| 404 | Not Found | Ressource non trouvée |
| 409 | Conflict | Conflit avec l'état actuel de la ressource |
| 413 | Payload Too Large | Taille de requête trop importante |
| 422 | Unprocessable Entity | Données valides mais non traitables |
| 429 | Too Many Requests | Limite de taux dépassée |
| 500 | Internal Server Error | Erreur interne du serveur |
| 502 | Bad Gateway | Erreur de communication avec service backend |
| 503 | Service Unavailable | Service temporairement indisponible |

## 🗄️ Stratégies de Cache

### TTL par Type de Données
```json
{
  "authentication": {
    "jwt_tokens": 900,        // 15 minutes
    "user_sessions": 86400,   // 24 heures
    "refresh_tokens": 604800  // 7 jours
  },
  "user_data": {
    "user_profile": 300,      // 5 minutes
    "user_preferences": 3600, // 1 heure
    "user_statistics": 1800   // 30 minutes
  },
  "project_data": {
    "project_list": 60,       // 1 minute
    "project_details": 300,   // 5 minutes
    "project_content": 1800,  // 30 minutes
    "project_versions": 3600  // 1 heure
  },
  "ai_results": {
    "analysis_results": 3600, // 1 heure
    "generation_status": 10,  // 10 secondes
    "model_performance": 7200 // 2 heures
  },
  "storage_data": {
    "file_metadata": 1800,    // 30 minutes
    "cdn_urls": 86400,        // 24 heures
    "thumbnails": 604800,     // 7 jours
    "transformation_status": 30 // 30 secondes
  },
  "static_content": {
    "api_documentation": 86400,  // 24 heures
    "configuration": 3600,       // 1 heure
    "feature_flags": 300         // 5 minutes
  }
}
```

### Stratégies d'Invalidation
```json
{
  "proactive": {
    "description": "Invalidation immédiate lors des modifications",
    "triggers": ["create", "update", "delete"],
    "examples": ["user_profile", "project_details"]
  },
  "reactive": {
    "description": "TTL court pour données critiques",
    "ttl_range": "10s - 5min",
    "examples": ["generation_status", "project_list"]
  },
  "lazy": {
    "description": "TTL long pour données statiques",
    "ttl_range": "1h - 7d",
    "examples": ["thumbnails", "cdn_urls", "api_documentation"]
  }
}
```

### Clés de Cache Standards
```
# Format: service:type:identifier:version
user:profile:{user_id}:v1
project:details:{project_id}:v1
storage:metadata:{file_id}:v1
ai:analysis:{job_id}:v1

# Format pour listes: service:type:filter:page:version
project:list:{user_id}:page1:v1
storage:files:{project_id}:page1:v1
```

## 🔄 Versioning API

### Convention de Versioning
- **URL** : `/api/v1/`, `/api/v2/`, etc.
- **Headers** : `Accept: application/vnd.visiobook.v1+json`
- **Rétrocompatibilité** : 12 mois minimum

### Stratégie de Migration
```json
{
  "phases": {
    "announcement": {
      "duration": "3 mois",
      "actions": ["documentation", "communication", "deprecation_warnings"]
    },
    "coexistence": {
      "duration": "6 mois",
      "actions": ["parallel_versions", "migration_tools", "monitoring"]
    },
    "migration": {
      "duration": "3 mois",
      "actions": ["forced_migration", "support_legacy", "cleanup"]
    }
  }
}
```

### Headers de Versioning
```http
# Requête
Accept: application/vnd.visiobook.v1+json
API-Version: v1

# Réponse
API-Version: v1
Deprecation: true
Sunset: 2024-12-31T23:59:59Z
Link: </api/v2/endpoint>; rel="successor-version"
```

### Format de Migration
```json
{
  "migration": {
    "from_version": "v1",
    "to_version": "v2",
    "breaking_changes": false,
    "changes": [
      {
        "type": "field_added|field_removed|field_renamed|endpoint_changed",
        "description": "Description of the change",
        "impact": "low|medium|high",
        "migration_guide": "URL to migration documentation"
      }
    ],
    "migration_deadline": "2024-12-31T23:59:59Z",
    "support_contact": "api-support@visiobook.com"
  }
}
```

## 📊 Monitoring et Observabilité

### Métriques Standards
```json
{
  "performance": {
    "response_time_ms": "Temps de réponse en millisecondes",
    "throughput_rps": "Requêtes par seconde",
    "error_rate_percent": "Taux d'erreur en pourcentage"
  },
  "business": {
    "active_users": "Utilisateurs actifs",
    "api_calls_count": "Nombre d'appels API",
    "feature_usage": "Utilisation des fonctionnalités"
  },
  "infrastructure": {
    "cpu_usage_percent": "Utilisation CPU",
    "memory_usage_mb": "Utilisation mémoire",
    "disk_usage_percent": "Utilisation disque"
  }
}
```

### Logs Standards
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info|warn|error|debug",
  "service": "service_name",
  "request_id": "req_123456789",
  "user_id": "user_123456789",
  "endpoint": "/api/v1/endpoint",
  "method": "GET|POST|PUT|DELETE",
  "status_code": 200,
  "response_time_ms": 150,
  "message": "Human readable message",
  "metadata": {
    "additional": "context"
  }
}
```

## 🔒 Sécurité

### Rate Limiting
```json
{
  "global": {
    "requests_per_minute": 1000,
    "burst_limit": 100
  },
  "per_user": {
    "requests_per_minute": 100,
    "burst_limit": 20
  },
  "per_endpoint": {
    "auth_endpoints": 10,
    "upload_endpoints": 5,
    "ai_endpoints": 2
  }
}
```

### Validation des Données
```json
{
  "input_validation": {
    "sanitization": "Nettoyage des entrées utilisateur",
    "type_checking": "Vérification des types de données",
    "size_limits": "Limites de taille des données",
    "format_validation": "Validation des formats (email, URL, etc.)"
  },
  "output_validation": {
    "data_masking": "Masquage des données sensibles",
    "response_filtering": "Filtrage des réponses selon permissions",
    "error_sanitization": "Nettoyage des messages d'erreur"
  }
}
```

## 📝 Documentation

### Standards de Documentation
- **Format** : Markdown avec diagrammes Mermaid
- **Structure** : Suivre le template de ce document
- **Mise à jour** : Synchronisée avec les changements d'API
- **Validation** : Revue obligatoire avant publication

### Références
- **Spécification OpenAPI** : Génération automatique depuis le code
- **Exemples** : Cas d'usage réels avec données d'exemple
- **Guides** : Tutoriels pas-à-pas pour intégration
- **Changelog** : Historique des modifications

---

**Version** : 1.0.0
**Dernière mise à jour** : 2024-01-15
**Contact** : architecture@visiobook.com

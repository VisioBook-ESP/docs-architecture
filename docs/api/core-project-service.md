# API Documentation - Core Project Service

## Vue d'ensemble du service

### Rôle et responsabilités
Le **Core Project Service** est responsable de la gestion complète des projets utilisateur, incluant la création, l'édition, le versioning, la collaboration et l'orchestration des workflows de génération. Il constitue le cœur de la logique métier de Visiobook.

### Justification de l'atomisation
- **Logique métier centralisée** : Gestion complexe des états et workflows de projets
- **Orchestration** : Coordination entre les services IA, stockage et utilisateurs
- **Évolutivité** : Fonctionnalités projet peuvent évoluer indépendamment
- **Performance** : Optimisation spécifique pour les requêtes fréquentes de projets

### Informations techniques
- **Port** : 8086
- **Technology Stack** : Node.js 18 + NestJS + TypeScript + Bull Queue
- **Database** : PostgreSQL + MongoDB + Redis (cache)
- **State Management** : State Machines + Event Sourcing
- **Version API** : v1

## Architecture du service

```mermaid
graph TB
    subgraph "Core Project Service"
        API[API Layer<br/>NestJS + TypeScript]
        PROJECT[Project Manager<br/>CRUD + Lifecycle]
        WORKFLOW[Workflow Engine<br/>State Machines]
        COLLAB[Collaboration Module<br/>Real-time + Permissions]
        VERSION[Version Control<br/>Git-like Versioning]
        QUEUE[Job Queue<br/>Bull + Redis]
    end

    subgraph "External Dependencies"
        USER_SVC[Core User Service<br/>Authentication]
        STORAGE_SVC[Storage Service<br/>File Management]
        AI_SVC[AI Analysis Service<br/>Content Processing]
        DB[(PostgreSQL<br/>Project Data)]
        MONGO[(MongoDB<br/>Content + Metadata)]
        REDIS[(Redis<br/>Cache + Queue)]
    end

    API --> PROJECT
    API --> WORKFLOW
    API --> COLLAB
    API --> VERSION
    PROJECT --> QUEUE
    WORKFLOW --> QUEUE

    PROJECT --> DB
    PROJECT --> MONGO
    COLLAB --> REDIS
    VERSION --> DB
    QUEUE --> REDIS

    API --> USER_SVC
    API --> STORAGE_SVC
    WORKFLOW --> AI_SVC

    classDef service fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef external fill:#fff8e1,stroke:#f9a825,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#388e3c,stroke-width:2px

    class API,PROJECT,WORKFLOW,COLLAB,VERSION,QUEUE service
    class USER_SVC,STORAGE_SVC,AI_SVC external
    class DB,MONGO,REDIS database
```

### Schémas de base de données

#### PostgreSQL - Tables principales
```sql
-- Projects table
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
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

CREATE INDEX idx_projects_user_id ON projects(user_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_visibility ON projects(visibility);
CREATE INDEX idx_projects_created_at ON projects(created_at);

-- Project versions table
CREATE TABLE project_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    version_number INTEGER NOT NULL,
    version_name VARCHAR(100),
    description TEXT,
    changes JSONB NOT NULL,
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(project_id, version_number)
);

CREATE INDEX idx_project_versions_project_id ON project_versions(project_id);
CREATE INDEX idx_project_versions_number ON project_versions(project_id, version_number);

-- Project collaborators table
CREATE TABLE project_collaborators (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    user_id UUID NOT NULL,
    role VARCHAR(50) NOT NULL,
    permissions JSONB DEFAULT '{}',
    invited_by UUID NOT NULL,
    invited_at TIMESTAMP DEFAULT NOW(),
    accepted_at TIMESTAMP,
    status VARCHAR(50) DEFAULT 'pending'
);

CREATE INDEX idx_project_collaborators_project_id ON project_collaborators(project_id);
CREATE INDEX idx_project_collaborators_user_id ON project_collaborators(user_id);
CREATE INDEX idx_project_collaborators_status ON project_collaborators(status);

-- Project workflows table
CREATE TABLE project_workflows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    workflow_type VARCHAR(100) NOT NULL,
    current_step VARCHAR(100) NOT NULL,
    steps_completed JSONB DEFAULT '[]',
    steps_remaining JSONB DEFAULT '[]',
    progress_percent INTEGER DEFAULT 0,
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    error_message TEXT,
    status VARCHAR(50) DEFAULT 'running'
);

CREATE INDEX idx_project_workflows_project_id ON project_workflows(project_id);
CREATE INDEX idx_project_workflows_status ON project_workflows(status);
CREATE INDEX idx_project_workflows_type ON project_workflows(workflow_type);

-- Project comments table
CREATE TABLE project_comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    user_id UUID NOT NULL,
    parent_comment_id UUID REFERENCES project_comments(id),
    content TEXT NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

CREATE INDEX idx_project_comments_project_id ON project_comments(project_id);
CREATE INDEX idx_project_comments_user_id ON project_comments(user_id);
CREATE INDEX idx_project_comments_parent ON project_comments(parent_comment_id);
```

#### MongoDB - Collections de contenu
```javascript
// projects_content collection
{
  _id: ObjectId,
  project_id: "uuid",
  content_type: "source|analysis|generated",
  version: 1,
  data: {
    // Structure flexible selon le type de contenu
    text: "contenu textuel",
    scenes: [...],
    characters: [...],
    settings: {...}
  },
  metadata: {
    word_count: 1500,
    language: "fr",
    genre: "fiction"
  },
  created_at: ISODate,
  updated_at: ISODate
}

// project_analytics collection
{
  _id: ObjectId,
  project_id: "uuid",
  event_type: "view|edit|generate|share",
  user_id: "uuid",
  data: {...},
  timestamp: ISODate
}
```

### Variables d'environnement

```bash
# Database connections
DATABASE_URL=postgresql://user:password@localhost:5432/visiobook
MONGODB_URL=mongodb://localhost:27017/visiobook
REDIS_URL=redis://localhost:6379

# External services
USER_SERVICE_URL=http://core-user-service:8081
STORAGE_SERVICE_URL=http://support-storage-service:8089
AI_SERVICE_URL=http://ai-analysis-service:8083

# Queue configuration
QUEUE_REDIS_URL=redis://localhost:6379
QUEUE_CONCURRENCY=5
QUEUE_RETRY_ATTEMPTS=3
QUEUE_RETRY_DELAY=5000

# Workflow settings
MAX_WORKFLOW_DURATION=3600
WORKFLOW_CLEANUP_INTERVAL=86400
AUTO_SAVE_INTERVAL=30

# Collaboration
REALTIME_ENABLED=true
MAX_COLLABORATORS=10
COMMENT_MAX_LENGTH=1000

# Security
JWT_SECRET=your-jwt-secret
ENCRYPTION_KEY=your-encryption-key

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=info
```

## Authentification et sécurité

### Système JWT
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json",
  "X-Project-Version": "1.2.3"
}
```

### Niveaux de permissions
- **admin** : Accès complet à tous les projets et gestion système
- **user** : Accès aux projets personnels et projets partagés selon permissions
- **premium** : Fonctionnalités avancées et quotas étendus

### Permissions de collaboration
- **owner** : Contrôle total du projet
- **editor** : Modification du contenu et des paramètres
- **viewer** : Lecture seule et commentaires
- **commenter** : Ajout de commentaires uniquement

### Headers de sécurité requis
```http
Authorization: Bearer <jwt_token>
X-Request-ID: <unique_request_id>
X-Client-Version: <client_version>
X-Project-Lock: <lock_token>
```

## Endpoints API

### Health & Monitoring

#### GET /health
**Description** : Vérification de l'état du service projet

**Permissions** : Aucune

**Réponse** :
```json
{
  "status": "UP",
  "timestamp": "2024-01-15T10:30:00Z",
  "service": "core-project-service",
  "version": "1.0.0",
  "checks": {
    "database": {
      "status": "UP",
      "responseTime": "15ms",
      "connections": 8
    },
    "mongodb": {
      "status": "UP",
      "responseTime": "12ms",
      "collections": 3
    },
    "redis": {
      "status": "UP",
      "responseTime": "3ms",
      "memory_usage": "45MB"
    },
    "queue": {
      "status": "UP",
      "pending_jobs": 12,
      "active_workers": 5
    }
  }
}
```

### Project Management

#### GET /api/v1/projects
**Description** : Liste des projets de l'utilisateur avec filtres

**Permissions** : user, premium, admin

**Paramètres de requête** :
```
?status=draft,active
&visibility=private,shared
&page=1
&limit=20
&sort=updated_at
&order=desc
&search=mon projet
&collaborator=user_id
```

**Réponse** :
```json
{
  "projects": [
    {
      "id": "proj_123456789",
      "title": "Mon Premier Livre",
      "description": "Une histoire captivante sur...",
      "status": "active",
      "visibility": "private",
      "progress": {
        "current_step": "ai_analysis",
        "progress_percent": 65,
        "estimated_completion": "2024-01-15T12:00:00Z"
      },
      "collaborators_count": 3,
      "created_at": "2024-01-10T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "owner": {
        "id": "user_123456789",
        "username": "johndoe",
        "avatar_url": "https://cdn.visiobook.com/avatars/user_123.jpg"
      },
      "stats": {
        "views": 45,
        "generations": 3,
        "comments": 8
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "pages": 8
  },
  "summary": {
    "total_projects": 156,
    "active_projects": 23,
    "completed_projects": 45,
    "draft_projects": 88
  }
}
```

#### POST /api/v1/projects
**Description** : Création d'un nouveau projet

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "title": "Mon Nouveau Projet",
  "description": "Description du projet",
  "visibility": "private",
  "source_content": {
    "type": "text",
    "content": "Il était une fois...",
    "metadata": {
      "language": "fr",
      "genre": "fiction"
    }
  },
  "settings": {
    "style": "realistic",
    "duration": "medium",
    "voice_language": "fr",
    "auto_generate": true
  },
  "collaborators": [
    {
      "user_id": "user_987654321",
      "role": "editor"
    }
  ]
}
```

**Réponse** :
```json
{
  "project": {
    "id": "proj_123456789",
    "title": "Mon Nouveau Projet",
    "description": "Description du projet",
    "status": "draft",
    "visibility": "private",
    "source_content_id": "content_abc123",
    "settings": {
      "style": "realistic",
      "duration": "medium",
      "voice_language": "fr",
      "auto_generate": true
    },
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  },
  "workflow": {
    "id": "workflow_456789",
    "status": "queued",
    "next_step": "content_analysis"
  },
  "collaborators": [
    {
      "user_id": "user_987654321",
      "role": "editor",
      "status": "pending"
    }
  ]
}
```

#### GET /api/v1/projects/{project_id}
**Description** : Détails complets d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Réponse** :
```json
{
  "project": {
    "id": "proj_123456789",
    "title": "Mon Premier Livre",
    "description": "Une histoire captivante sur...",
    "status": "active",
    "visibility": "private",
    "source_content_id": "content_abc123",
    "generated_content_id": "content_def456",
    "settings": {
      "style": "realistic",
      "duration": "medium",
      "voice_language": "fr",
      "auto_generate": true,
      "quality": "high"
    },
    "metadata": {
      "word_count": 15420,
      "estimated_duration": "12 minutes",
      "genre": "fiction",
      "target_audience": "adult"
    },
    "created_at": "2024-01-10T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  },
  "content": {
    "source": {
      "type": "text",
      "content": "Il était une fois...",
      "metadata": {
        "language": "fr",
        "word_count": 15420
      }
    },
    "analysis": {
      "scenes": [
        {
          "id": "scene_1",
          "title": "Introduction",
          "description": "Présentation du héros",
          "characters": ["hero", "mentor"],
          "location": "village",
          "mood": "peaceful"
        }
      ],
      "characters": [
        {
          "id": "hero",
          "name": "Jean",
          "description": "Jeune homme courageux",
          "traits": ["brave", "curious"]
        }
      ]
    }
  },
  "workflow": {
    "id": "workflow_456789",
    "status": "completed",
    "current_step": "generation_complete",
    "progress_percent": 100,
    "steps_completed": [
      "content_analysis",
      "scene_extraction",
      "character_analysis",
      "image_generation",
      "animation_creation",
      "audio_synthesis"
    ],
    "completed_at": "2024-01-15T10:30:00Z"
  },
  "collaborators": [
    {
      "user_id": "user_987654321",
      "username": "janedoe",
      "role": "editor",
      "permissions": ["read", "write", "comment"],
      "status": "accepted",
      "last_activity": "2024-01-15T09:45:00Z"
    }
  ],
  "versions": {
    "current": 3,
    "total": 5,
    "latest_changes": "Amélioration des dialogues"
  },
  "stats": {
    "views": 45,
    "generations": 3,
    "comments": 8,
    "shares": 2
  }
}
```

#### PUT /api/v1/projects/{project_id}
**Description** : Mise à jour d'un projet

**Permissions** : user, premium, admin (propriétaire, editor ou admin)

**Requête** :
```json
{
  "title": "Mon Livre Mis à Jour",
  "description": "Description mise à jour",
  "visibility": "shared",
  "settings": {
    "style": "cartoon",
    "duration": "long",
    "quality": "ultra"
  },
  "version_info": {
    "version_name": "v1.2",
    "description": "Amélioration du style visuel"
  }
}
```

**Réponse** :
```json
{
  "project": {
    "id": "proj_123456789",
    "title": "Mon Livre Mis à Jour",
    "description": "Description mise à jour",
    "visibility": "shared",
    "updated_at": "2024-01-15T11:30:00Z"
  },
  "version": {
    "number": 4,
    "name": "v1.2",
    "description": "Amélioration du style visuel",
    "created_at": "2024-01-15T11:30:00Z"
  },
  "workflow": {
    "id": "workflow_789123",
    "status": "queued",
    "message": "Regeneration queued due to settings change"
  }
}
```

#### DELETE /api/v1/projects/{project_id}
**Description** : Suppression d'un projet

**Permissions** : user, premium, admin (propriétaire ou admin)

**Paramètres de requête** :
```
?permanent=false
```

**Réponse** :
```json
{
  "project_id": "proj_123456789",
  "deleted": true,
  "permanent": false,
  "recovery_deadline": "2024-02-14T10:30:00Z",
  "cleanup_scheduled": "2024-02-14T10:30:00Z",
  "message": "Project moved to trash. Can be recovered within 30 days."
}
```

### Content Management

#### GET /api/v1/projects/{project_id}/content
**Description** : Récupération du contenu d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Paramètres de requête** :
```
?type=source,analysis,generated
&version=3
&format=json,text
```

**Réponse** :
```json
{
  "content": {
    "source": {
      "type": "text",
      "content": "Il était une fois...",
      "metadata": {
        "language": "fr",
        "word_count": 15420,
        "reading_time": "45 minutes"
      }
    },
    "analysis": {
      "summary": "Histoire d'aventure avec un jeune héros...",
      "scenes": [
        {
          "id": "scene_1",
          "title": "Le départ",
          "content": "Jean quitte son village...",
          "characters": ["Jean", "Mentor"],
          "location": "Village de Lumière",
          "mood": "nostalgique",
          "duration_estimate": "2 minutes"
        }
      ],
      "characters": [
        {
          "id": "jean",
          "name": "Jean",
          "description": "Jeune homme de 20 ans, courageux et déterminé",
          "appearance": "Cheveux bruns, yeux verts, taille moyenne",
          "personality": ["courageux", "curieux", "loyal"],
          "role": "protagonist"
        }
      ],
      "themes": ["courage", "amitié", "découverte"],
      "genre": "fantasy",
      "target_audience": "young_adult"
    },
    "generated": {
      "storyboard": [
        {
          "scene_id": "scene_1",
          "shots": [
            {
              "id": "shot_1",
              "description": "Plan large du village au lever du soleil",
              "image_url": "https://cdn.visiobook.com/generated/shot_1.jpg",
              "duration": 3.5
            }
          ]
        }
      ],
      "audio": {
        "narration_url": "https://cdn.visiobook.com/audio/narration.mp3",
        "music_url": "https://cdn.visiobook.com/audio/background.mp3",
        "effects": []
      },
      "final_video": {
        "url": "https://cdn.visiobook.com/videos/final.mp4",
        "duration": 720,
        "resolution": "1920x1080",
        "format": "mp4"
      }
    }
  },
  "version": 3,
  "last_updated": "2024-01-15T10:30:00Z"
}
```

#### PUT /api/v1/projects/{project_id}/content
**Description** : Mise à jour du contenu source d'un projet

**Permissions** : user, premium, admin (propriétaire, editor ou admin)

**Requête** :
```json
{
  "content": "Il était une fois, dans un royaume lointain...",
  "metadata": {
    "language": "fr",
    "genre": "fantasy"
  },
  "auto_regenerate": true,
  "version_info": {
    "description": "Ajout du prologue"
  }
}
```

**Réponse** :
```json
{
  "content_updated": true,
  "version": {
    "number": 4,
    "description": "Ajout du prologue",
    "created_at": "2024-01-15T11:30:00Z"
  },
  "workflow": {
    "id": "workflow_new123",
    "status": "queued",
    "estimated_completion": "2024-01-15T12:15:00Z"
  },
  "changes": {
    "word_count_delta": 250,
    "new_word_count": 15670,
    "scenes_affected": ["scene_1"]
  }
}
```

### Workflow Management

#### GET /api/v1/projects/{project_id}/workflows
**Description** : Liste des workflows d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Réponse** :
```json
{
  "workflows": [
    {
      "id": "workflow_123456789",
      "type": "full_generation",
      "status": "completed",
      "current_step": "generation_complete",
      "progress_percent": 100,
      "started_at": "2024-01-15T10:00:00Z",
      "completed_at": "2024-01-15T10:30:00Z",
      "duration_ms": 1800000,
      "steps": [
        {
          "name": "content_analysis",
          "status": "completed",
          "duration_ms": 45000,
          "result": "success"
        },
        {
          "name": "scene_extraction",
          "status": "completed",
          "duration_ms": 120000,
          "result": "success"
        },
        {
          "name": "image_generation",
          "status": "completed",
          "duration_ms": 900000,
          "result": "success"
        }
      ]
    }
  ],
  "current_workflow": {
    "id": "workflow_987654321",
    "status": "running",
    "progress_percent": 45,
    "estimated_completion": "2024-01-15T11:45:00Z"
  }
}
```

#### POST /api/v1/projects/{project_id}/workflows
**Description** : Démarrage d'un nouveau workflow

**Permissions** : user, premium, admin (propriétaire, editor ou admin)

**Requête** :
```json
{
  "workflow_type": "regenerate_images",
  "parameters": {
    "style": "cartoon",
    "quality": "high",
    "scenes": ["scene_1", "scene_3"]
  },
  "priority": "normal"
}
```

**Réponse** :
```json
{
  "workflow": {
    "id": "workflow_new456",
    "type": "regenerate_images",
    "status": "queued",
    "queue_position": 3,
    "estimated_start": "2024-01-15T11:35:00Z",
    "estimated_completion": "2024-01-15T12:00:00Z",
    "parameters": {
      "style": "cartoon",
      "quality": "high",
      "scenes": ["scene_1", "scene_3"]
    }
  },
  "cost_estimate": {
    "credits": 150,
    "currency": "EUR",
    "amount": 15.00
  }
}
```

#### GET /api/v1/projects/{project_id}/workflows/{workflow_id}
**Description** : Détails d'un workflow spécifique

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Réponse** :
```json
{
  "workflow": {
    "id": "workflow_123456789",
    "project_id": "proj_123456789",
    "type": "full_generation",
    "status": "running",
    "current_step": "image_generation",
    "progress_percent": 65,
    "started_at": "2024-01-15T10:00:00Z",
    "estimated_completion": "2024-01-15T11:30:00Z",
    "parameters": {
      "style": "realistic",
      "quality": "high",
      "voice_language": "fr"
    }
  },
  "steps": [
    {
      "name": "content_analysis",
      "status": "completed",
      "started_at": "2024-01-15T10:00:00Z",
      "completed_at": "2024-01-15T10:02:00Z",
      "duration_ms": 120000,
      "result": {
        "scenes_extracted": 12,
        "characters_identified": 5,
        "confidence_score": 0.92
      }
    },
    {
      "name": "image_generation",
      "status": "running",
      "started_at": "2024-01-15T10:02:00Z",
      "progress_percent": 65,
      "current_task": "Generating scene 8 of 12",
      "estimated_completion": "2024-01-15T10:15:00Z"
    }
  ],
  "logs": [
    {
      "timestamp": "2024-01-15T10:00:00Z",
      "level": "info",
      "message": "Workflow started",
      "step": "initialization"
    },
    {
      "timestamp": "2024-01-15T10:02:00Z",
      "level": "info",
      "message": "Content analysis completed successfully",
      "step": "content_analysis"
    }
  ],
  "resource_usage": {
    "cpu_time_ms": 45000,
    "memory_peak_mb": 2048,
    "gpu_time_ms": 180000,
    "storage_used_mb": 1024
  }
}
```

### Collaboration

#### GET /api/v1/projects/{project_id}/collaborators
**Description** : Liste des collaborateurs d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Réponse** :
```json
{
  "collaborators": [
    {
      "user_id": "user_123456789",
      "username": "johndoe",
      "email": "john@example.com",
      "role": "owner",
      "permissions": ["read", "write", "delete", "manage_collaborators"],
      "status": "active",
      "joined_at": "2024-01-10T10:30:00Z",
      "last_activity": "2024-01-15T10:30:00Z",
      "avatar_url": "https://cdn.visiobook.com/avatars/user_123.jpg"
    },
    {
      "user_id": "user_987654321",
      "username": "janedoe",
      "email": "jane@example.com",
      "role": "editor",
      "permissions": ["read", "write", "comment"],
      "status": "accepted",
      "invited_by": "user_123456789",
      "invited_at": "2024-01-12T14:20:00Z",
      "accepted_at": "2024-01-12T15:30:00Z",
      "last_activity": "2024-01-15T09:45:00Z"
    }
  ],
  "pending_invitations": [
    {
      "email": "bob@example.com",
      "role": "viewer",
      "invited_by": "user_123456789",
      "invited_at": "2024-01-14T16:00:00Z",
      "expires_at": "2024-01-21T16:00:00Z"
    }
  ],
  "collaboration_settings": {
    "max_collaborators": 10,
    "allow_public_comments": false,
    "require_approval": true
  }
}
```

#### POST /api/v1/projects/{project_id}/collaborators
**Description** : Invitation d'un collaborateur

**Permissions** : user, premium, admin (propriétaire ou admin)

**Requête** :
```json
{
  "email": "newuser@example.com",
  "role": "editor",
  "permissions": ["read", "write", "comment"],
  "message": "J'aimerais que tu m'aides sur ce projet"
}
```

**Réponse** :
```json
{
  "invitation": {
    "id": "invite_123456789",
    "email": "newuser@example.com",
    "role": "editor",
    "permissions": ["read", "write", "comment"],
    "invited_by": "user_123456789",
    "invited_at": "2024-01-15T10:30:00Z",
    "expires_at": "2024-01-22T10:30:00Z",
    "status": "sent"
  },
  "notification_sent": true,
  "message": "Invitation sent successfully"
}
```

#### PUT /api/v1/projects/{project_id}/collaborators/{user_id}
**Description** : Modification du rôle d'un collaborateur

**Permissions** : user, premium, admin (propriétaire ou admin)

**Requête** :
```json
{
  "role": "viewer",
  "permissions": ["read", "comment"]
}
```

**Réponse** :
```json
{
  "collaborator": {
    "user_id": "user_987654321",
    "role": "viewer",
    "permissions": ["read", "comment"],
    "updated_at": "2024-01-15T11:30:00Z"
  },
  "message": "Collaborator role updated successfully"
}
```

#### DELETE /api/v1/projects/{project_id}/collaborators/{user_id}
**Description** : Suppression d'un collaborateur

**Permissions** : user, premium, admin (propriétaire ou admin)

**Réponse** :
```json
{
  "removed": true,
  "user_id": "user_987654321",
  "message": "Collaborator removed from project"
}
```

### Comments

#### GET /api/v1/projects/{project_id}/comments
**Description** : Liste des commentaires d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Paramètres de requête** :
```
?page=1
&limit=20
&sort=created_at
&order=desc
&thread=comment_id
```

**Réponse** :
```json
{
  "comments": [
    {
      "id": "comment_123456789",
      "user_id": "user_987654321",
      "username": "janedoe",
      "avatar_url": "https://cdn.visiobook.com/avatars/user_987.jpg",
      "content": "J'adore cette scène ! Peut-être ajouter plus de détails sur le paysage ?",
      "parent_comment_id": null,
      "replies_count": 2,
      "metadata": {
        "scene_id": "scene_1",
        "timestamp": "00:02:15"
      },
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    },
    {
      "id": "comment_987654321",
      "user_id": "user_123456789",
      "username": "johndoe",
      "avatar_url": "https://cdn.visiobook.com/avatars/user_123.jpg",
      "content": "Bonne idée ! Je vais ajouter une description du coucher de soleil.",
      "parent_comment_id": "comment_123456789",
      "replies_count": 0,
      "metadata": {},
      "created_at": "2024-01-15T10:45:00Z",
      "updated_at": "2024-01-15T10:45:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "pages": 3
  }
}
```

#### POST /api/v1/projects/{project_id}/comments
**Description** : Ajout d'un commentaire

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Requête** :
```json
{
  "content": "Excellente progression dans cette scène !",
  "parent_comment_id": null,
  "metadata": {
    "scene_id": "scene_2",
    "timestamp": "00:05:30"
  }
}
```

**Réponse** :
```json
{
  "comment": {
    "id": "comment_new123",
    "user_id": "user_123456789",
    "content": "Excellente progression dans cette scène !",
    "parent_comment_id": null,
    "metadata": {
      "scene_id": "scene_2",
      "timestamp": "00:05:30"
    },
    "created_at": "2024-01-15T11:30:00Z"
  },
  "notifications_sent": 2
}
```

### Version Control

#### GET /api/v1/projects/{project_id}/versions
**Description** : Liste des versions d'un projet

**Permissions** : user, premium, admin (propriétaire, collaborateur ou admin)

**Réponse** :
```json
{
  "versions": [
    {
      "number": 3,
      "name": "v1.2",
      "description": "Amélioration des dialogues",
      "created_by": "user_123456789",
      "created_by_username": "johndoe",
      "created_at": "2024-01-15T10:30:00Z",
      "changes": {
        "content_modified": true,
        "settings_modified": false,
        "scenes_added": 1,
        "scenes_modified": 3,
        "word_count_delta": 250
      },
      "is_current": true
    },
    {
      "number": 2,
      "name": "v1.1",
      "description": "Correction des personnages",
      "created_by": "user_987654321",
      "created_by_username": "janedoe",
      "created_at": "2024-01-14T15:20:00Z",
      "changes": {
        "content_modified": true,
        "settings_modified": true,
        "scenes_added": 0,
        "scenes_modified": 2,
        "word_count_delta": -150
      },
      "is_current": false
    }
  ],
  "current_version": 3,
  "total_versions": 5
}
```

#### POST /api/v1/projects/{project_id}/versions/{version_number}/restore
**Description** : Restauration d'une version antérieure

**Permissions** : user, premium, admin (propriétaire, editor ou admin)

**Réponse** :
```json
{
  "restored": true,
  "new_version": {
    "number": 4,
    "name": "Restored from v1.1",
    "description": "Restoration of version 2",
    "created_at": "2024-01-15T11:30:00Z"
  },
  "workflow": {
    "id": "workflow_restore123",
    "status": "queued",
    "message": "Regeneration queued for restored content"
  }
}
```

## Flux de transactions CRUD

### Diagramme de séquence - Création de projet

```mermaid
sequenceDiagram
    participant Client as Web Client
    participant API as Project API
    participant UserSvc as User Service
    participant StorageSvc as Storage Service
    participant AISvc as AI Service
    participant DB as Database
    participant Queue as Job Queue

    Client->>API: POST /api/v1/projects
    API->>UserSvc: Validate user permissions
    UserSvc-->>API: User validated
    API->>DB: Create project record
    DB-->>API: Project created
    API->>StorageSvc: Store source content
    StorageSvc-->>API: Content stored
    API->>Queue: Queue analysis workflow
    Queue-->>API: Workflow queued
    API->>AISvc: Start content analysis
    AISvc-->>API: Analysis started
    API-->>Client: Return project + workflow info
```

### Diagramme de séquence - Collaboration en temps réel

```mermaid
sequenceDiagram
    participant User1 as User 1
    participant User2 as User 2
    participant API as Project API
    participant Redis as Redis Cache
    participant WebSocket as WebSocket Server

    User1->>API: PUT /api/v1/projects/{id}/content
    API->>Redis: Update project cache
    API->>WebSocket: Broadcast content change
    WebSocket->>User2: Real-time update notification
    User2->>API: GET /api/v1/projects/{id}/content
    API->>Redis: Get latest content
    Redis-->>API: Return cached content
    API-->>User2: Return updated content
```

### Diagramme de flux - Workflow de génération

```mermaid
flowchart TD
    Start([Project Creation]) --> ValidateContent{Valid Content?}
    ValidateContent -->|No| ContentError[Return Content Error]
    ValidateContent -->|Yes| QueueAnalysis[Queue Content Analysis]

    QueueAnalysis --> AnalysisStep[AI Content Analysis]
    AnalysisStep --> CheckAnalysis{Analysis Success?}
    CheckAnalysis -->|No| AnalysisError[Analysis Failed]
    CheckAnalysis -->|Yes| ExtractScenes[Extract Scenes]

    ExtractScenes --> GenerateImages[Generate Scene Images]
    GenerateImages --> CreateAnimations[Create Animations]
    CreateAnimations --> SynthesizeAudio[Synthesize Audio]
    SynthesizeAudio --> AssembleVideo[Assemble Final Video]

    AssembleVideo --> UpdateProject[Update Project Status]
    UpdateProject --> NotifyUsers[Notify Collaborators]
    NotifyUsers --> Success[Generation Complete]

    ContentError --> End([End])
    AnalysisError --> End
    Success --> End
```

## Parcours utilisateurs détaillés

### Milestone 1: Importer un contenu

**US 1.1 - Import de fichiers**
```
1. POST /api/v1/projects
   - Création du projet avec contenu source
   - Démarrage automatique du workflow d'analyse

2. GET /api/v1/projects/{project_id}/workflows
   - Suivi de la progression d'analyse
   - Notification de fin de traitement
```

**US 1.3 - Prévisualisation et résumé**
```
1. GET /api/v1/projects/{project_id}/content?type=analysis
   - Récupération du résumé généré par l'IA
   - Affichage des scènes extraites

2. PUT /api/v1/projects/{project_id}/content
   - Modification du contenu si nécessaire
   - Relance de l'analyse si modifié
```

**US 1.4 - Validation du contenu**
```
1. GET /api/v1/projects/{project_id}/content?type=analysis
   - Récupération de l'analyse détaillée
   - Vérification des personnages et scènes

2. PUT /api/v1/projects/{project_id}
   - Validation et passage au statut "validated"
   - Déclenchement de la génération
```

### Milestone 2: Personnaliser le style de l'animation

**US 2.1 - Choix du style graphique**
```
1. PUT /api/v1/projects/{project_id}
   - Mise à jour des paramètres de style
   - Déclenchement de la régénération

2. POST /api/v1/projects/{project_id}/workflows
   - Lancement du workflow de régénération
   - Suivi de la progression
```

**US 2.3 - Définir la durée de l'animation**
```
1. PUT /api/v1/projects/{project_id}
   - Modification des paramètres de durée
   - Recalcul automatique des scènes

2. GET /api/v1/projects/{project_id}/content?type=analysis
   - Vérification de l'impact sur les scènes
   - Validation des changements
```

### Milestone 3: Générer et visualiser une animation

**US 3.1 - Génération automatique**
```
1. POST /api/v1/projects/{project_id}/workflows
   - Démarrage du workflow de génération complète
   - Configuration des paramètres de qualité

2. GET /api/v1/projects/{project_id}/workflows/{workflow_id}
   - Suivi en temps réel de la progression
   - Affichage des étapes en cours
```

**US 3.3 - Suivi de la progression**
```
1. GET /api/v1/projects/{project_id}/workflows
   - Récupération du statut des workflows actifs
   - Estimation du temps restant

2. WebSocket connection
   - Notifications en temps réel
   - Mise à jour automatique de l'interface
```

### Milestone 5: Historique et réutilisation

**US 5.1 - Accéder à l'historique**
```
1. GET /api/v1/projects
   - Liste de tous les projets utilisateur
   - Filtrage par statut et date

2. GET /api/v1/projects/{project_id}
   - Détails complets du projet sélectionné
   - Historique des modifications
```

**US 5.2 - Modifier un projet existant**
```
1. PUT /api/v1/projects/{project_id}
   - Modification des paramètres du projet
   - Création automatique d'une nouvelle version

2. GET /api/v1/projects/{project_id}/versions
   - Consultation de l'historique des versions
   - Possibilité de restauration
```

**US 5.3 - Dupliquer un projet**
```
1. POST /api/v1/projects (avec source_project_id)
   - Création d'un nouveau projet basé sur l'existant
   - Copie du contenu et des paramètres

2. PUT /api/v1/projects/{new_project_id}
   - Modification des paramètres du projet dupliqué
   - Personnalisation selon les besoins
```

## Codes d'erreur

| Code | Message | Description |
|------|---------|-------------|
| 200 | Success | Opération réussie |
| 201 | Created | Projet créé avec succès |
| 202 | Accepted | Workflow en cours de traitement |
| 400 | Bad Request | Données de projet invalides |
| 401 | Unauthorized | Token JWT manquant ou invalide |
| 403 | Forbidden | Permissions insuffisantes sur le projet |
| 404 | Not Found | Projet non trouvé |
| 409 | Conflict | Conflit de version ou de collaboration |
| 422 | Unprocessable Entity | Contenu invalide ou workflow impossible |
| 423 | Locked | Projet verrouillé par un autre utilisateur |
| 429 | Too Many Requests | Limite de projets ou workflows dépassée |
| 500 | Internal Server Error | Erreur interne du serveur |
| 503 | Service Unavailable | Service temporairement indisponible |

### Format d'erreur standardisé
```json
{
  "error": {
    "code": "PROJECT_QUOTA_EXCEEDED",
    "message": "Maximum number of projects reached for user subscription",
    "details": {
      "current_projects": 25,
      "max_projects": 25,
      "subscription_type": "premium"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_123456789"
  }
}
```

### Codes d'erreur spécifiques
```json
{
  "INVALID_CONTENT": "Project content is invalid or corrupted",
  "WORKFLOW_FAILED": "Project generation workflow failed",
  "COLLABORATION_LIMIT": "Maximum number of collaborators reached",
  "VERSION_CONFLICT": "Version conflict detected during update",
  "CONTENT_TOO_LARGE": "Project content exceeds maximum size limit",
  "INSUFFICIENT_CREDITS": "Not enough credits to complete operation",
  "PROJECT_LOCKED": "Project is currently being edited by another user",
  "INVALID_WORKFLOW": "Requested workflow type is not supported"
}
```

## Versioning et migration

### Convention v1
- **URL Base** : `/api/v1/`
- **Headers** : `Accept: application/vnd.visiobook.v1+json`
- **Rétrocompatibilité** : Maintenue pendant 12 mois minimum

### Stratégie de migration
1. **Projets existants** : Migration transparente des données
2. **Workflows** : Conversion automatique vers nouveaux types
3. **Collaborations** : Préservation des permissions existantes
4. **Versions** : Maintien de l'historique complet

### Exemple de migration v1 → v2
```json
{
  "migration": {
    "from_version": "v1",
    "to_version": "v2",
    "changes": [
      {
        "type": "workflow_enhanced",
        "description": "New AI models with improved quality"
      },
      {
        "type": "collaboration_improved",
        "description": "Real-time collaborative editing"
      }
    ],
    "breaking_changes": false,
    "migration_deadline": "2024-12-31"
  }
}
```

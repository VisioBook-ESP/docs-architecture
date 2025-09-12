# API Documentation - AI Analysis Service

## Vue d'ensemble du service

### Rôle et responsabilités
Le **AI Analysis Service** est le cœur de l'intelligence artificielle de Visiobook. Il est responsable de l'analyse sémantique des contenus, de l'extraction de scènes, de la génération d'images et de la création de contenu multimédia à partir de texte. Il constitue la valeur métier principale de la plateforme.

### Justification de l'atomisation
- **Spécialisation IA** : Logique complexe d'analyse nécessitant des ressources GPU dédiées
- **Évolutivité** : Modèles IA peuvent être mis à jour indépendamment
- **Performance** : Optimisation spécifique pour les workloads IA et GPU
- **Coût** : Gestion optimisée des ressources GPU coûteuses

### Informations techniques
- **Port** : 8083
- **Technology Stack** : Python 3.11 + FastAPI + PyTorch + Transformers
- **AI Frameworks** : PyTorch 2.1 + Hugging Face + LangChain
- **GPU** : NVIDIA CUDA 12.1 + cuDNN 8.9
- **Version API** : v1

## Architecture du service

```mermaid
graph TB
    subgraph "AI Analysis Service"
        API[API Layer<br/>FastAPI + Python]
        SEMANTIC[Semantic Analysis<br/>BERT + RoBERTa]
        SCENE[Scene Extraction<br/>Custom Transformer]
        CHARACTER[Character Analysis<br/>NER + Classification]
        IMAGE[Image Generation<br/>Stable Diffusion]
        AUDIO[Audio Synthesis<br/>TTS + Music Gen]
        QUEUE[Task Queue<br/>Celery + Redis]
    end

    subgraph "AI Models"
        NLP[NLP Models<br/>Transformers]
        VISION[Vision Models<br/>Diffusion + GAN]
        AUDIO_MODEL[Audio Models<br/>TTS + Music]
        CACHE[Model Cache<br/>GPU Memory]
    end

    subgraph "External Dependencies"
        STORAGE[Storage Service<br/>File Access]
        PROJECT[Project Service<br/>Content + Metadata]
        DB[(PostgreSQL<br/>Analysis Results)]
        VECTOR[(Vector DB<br/>Embeddings)]
        REDIS[(Redis<br/>Queue + Cache)]
    end

    API --> SEMANTIC
    API --> SCENE
    API --> CHARACTER
    API --> IMAGE
    API --> AUDIO
    API --> QUEUE

    SEMANTIC --> NLP
    SCENE --> NLP
    CHARACTER --> NLP
    IMAGE --> VISION
    AUDIO --> AUDIO_MODEL

    NLP --> CACHE
    VISION --> CACHE
    AUDIO_MODEL --> CACHE

    API --> STORAGE
    API --> PROJECT
    SEMANTIC --> DB
    SCENE --> VECTOR
    QUEUE --> REDIS

    classDef service fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef model fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef external fill:#fff8e1,stroke:#f9a825,stroke-width:2px

    class API,SEMANTIC,SCENE,CHARACTER,IMAGE,AUDIO,QUEUE service
    class NLP,VISION,AUDIO_MODEL,CACHE model
    class STORAGE,PROJECT,DB,VECTOR,REDIS external
```

### Schémas de base de données

#### Note sur l'architecture de données

> **🏗️ Responsabilité de ce service (Phase actuelle)**
>
> L'AI Analysis Service est **propriétaire** de toutes les données d'analyse IA, jobs de traitement, résultats d'analyse et métriques de performance des modèles. Il mocke localement les références aux projets (project_id) nécessaires à son fonctionnement.
>
> **🎯 Migration future**
>
> Lors de la centralisation via le Core Database Service, ce service fournira les migrations de référence pour toutes les tables liées à l'intelligence artificielle et au traitement de contenu.

#### PostgreSQL - Tables propriétaires

```sql
-- Analysis jobs table (PROPRIÉTAIRE - AI Analysis Service)
-- Cette table est la source de vérité pour tous les jobs d'analyse IA
CREATE TABLE analysis_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL, -- RÉFÉRENCE MOCKÉE vers Core Project Service
    job_type VARCHAR(100) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    priority INTEGER DEFAULT 5,
    parameters JSONB NOT NULL,
    progress_percent INTEGER DEFAULT 0,
    result JSONB,
    error_message TEXT,
    gpu_time_ms BIGINT DEFAULT 0,
    cpu_time_ms BIGINT DEFAULT 0,
    memory_peak_mb INTEGER DEFAULT 0,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_analysis_jobs_project_id ON analysis_jobs(project_id);
CREATE INDEX idx_analysis_jobs_status ON analysis_jobs(status);
CREATE INDEX idx_analysis_jobs_type ON analysis_jobs(job_type);
CREATE INDEX idx_analysis_jobs_priority ON analysis_jobs(priority);

-- Content analysis table
CREATE TABLE content_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL,
    content_hash VARCHAR(64) NOT NULL,
    analysis_type VARCHAR(100) NOT NULL,
    model_version VARCHAR(50) NOT NULL,
    confidence_score DECIMAL(4,3),
    result JSONB NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(content_hash, analysis_type, model_version)
);

CREATE INDEX idx_content_analysis_project_id ON content_analysis(project_id);
CREATE INDEX idx_content_analysis_hash ON content_analysis(content_hash);
CREATE INDEX idx_content_analysis_type ON content_analysis(analysis_type);
CREATE INDEX idx_content_analysis_confidence ON content_analysis(confidence_score);

-- Generated assets table
CREATE TABLE generated_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL,
    asset_type VARCHAR(100) NOT NULL,
    generation_params JSONB NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size_bytes BIGINT,
    generation_time_ms BIGINT,
    model_used VARCHAR(100),
    quality_score DECIMAL(4,3),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_generated_assets_project_id ON generated_assets(project_id);
CREATE INDEX idx_generated_assets_type ON generated_assets(asset_type);
CREATE INDEX idx_generated_assets_model ON generated_assets(model_used);

-- Model performance table
CREATE TABLE model_performance (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    model_name VARCHAR(100) NOT NULL,
    model_version VARCHAR(50) NOT NULL,
    operation_type VARCHAR(100) NOT NULL,
    avg_processing_time_ms BIGINT,
    avg_gpu_utilization DECIMAL(5,2),
    avg_memory_usage_mb INTEGER,
    success_rate DECIMAL(5,4),
    last_updated TIMESTAMP DEFAULT NOW(),
    UNIQUE(model_name, model_version, operation_type)
);

CREATE INDEX idx_model_performance_name ON model_performance(model_name);
CREATE INDEX idx_model_performance_operation ON model_performance(operation_type);
```

### Variables d'environnement

```bash
# GPU Configuration
CUDA_VISIBLE_DEVICES=0,1,2,3
NVIDIA_VISIBLE_DEVICES=all
CUDA_MEMORY_FRACTION=0.8
GPU_MEMORY_GROWTH=true

# Model Configuration
MODELS_PATH=/app/models
MODEL_CACHE_SIZE=8GB
MODEL_PRECISION=fp16
BATCH_SIZE_SEMANTIC=32
BATCH_SIZE_IMAGE=4

# Hugging Face
HF_TOKEN=your-huggingface-token
HF_CACHE_DIR=/app/cache/huggingface
TRANSFORMERS_CACHE=/app/cache/transformers

# OpenAI (optional)
OPENAI_API_KEY=your-openai-key
OPENAI_MODEL=gpt-4-turbo

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/visiobook
VECTOR_DB_URL=postgresql://user:password@localhost:5432/visiobook_vectors
REDIS_URL=redis://localhost:6379

# External Services
STORAGE_SERVICE_URL=http://support-storage-service:8089
PROJECT_SERVICE_URL=http://core-project-service:8086

# Queue Configuration
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1
CELERY_WORKERS=4
CELERY_CONCURRENCY=2

# Performance
MAX_CONTENT_LENGTH=52428800
MAX_PROCESSING_TIME=3600
TIMEOUT_SEMANTIC_ANALYSIS=300
TIMEOUT_IMAGE_GENERATION=600

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=info
ENABLE_PROFILING=true
```

## Authentification et sécurité

> **📋 Référence** : Voir [Règles Communes](./regles_communes.md) pour les standards d'authentification, permissions et sécurité.

### Système JWT
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json",
  "X-GPU-Priority": "high"
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
- **admin** : Accès complet aux modèles et gestion des ressources GPU
- **user** : Accès aux analyses selon quotas d'abonnement
- **premium** : Accès prioritaire et modèles avancés

### Headers de sécurité requis
```http
Authorization: Bearer <jwt_token>
X-Request-ID: <unique_request_id>
X-Client-Version: <client_version>
X-Content-Hash: <sha256_hash>
```

## Endpoints API

### Health & Monitoring

#### GET /health
**Description** : Vérification de l'état du service IA et des GPU

**Permissions** : Aucune

**Réponse** :
```json
{
  "status": "UP",
  "timestamp": "2024-01-15T10:30:00Z",
  "service": "ai-analysis-service",
  "version": "1.0.0",
  "checks": {
    "database": {
      "status": "UP",
      "responseTime": "18ms"
    },
    "redis": {
      "status": "UP",
      "responseTime": "3ms"
    },
    "gpu": {
      "status": "UP",
      "devices": [
        {
          "id": 0,
          "name": "NVIDIA RTX 4090",
          "memory_used": "12GB",
          "memory_total": "24GB",
          "utilization": 75,
          "temperature": 68
        }
      ]
    },
    "models": {
      "semantic_analysis": {
        "status": "LOADED",
        "model": "bert-large-uncased",
        "memory_usage": "2.1GB"
      },
      "image_generation": {
        "status": "LOADED",
        "model": "stable-diffusion-xl",
        "memory_usage": "6.8GB"
      }
    },
    "queue": {
      "status": "UP",
      "pending_jobs": 8,
      "active_workers": 4
    }
  }
}
```

### Content Analysis

#### POST /api/v1/analysis/semantic
**Description** : Analyse sémantique complète d'un contenu textuel

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "content": "Il était une fois, dans un royaume lointain, un jeune prince courageux...",
  "language": "fr",
  "analysis_options": {
    "extract_entities": true,
    "sentiment_analysis": true,
    "theme_detection": true,
    "genre_classification": true,
    "reading_level": true
  },
  "model_preferences": {
    "precision": "high",
    "speed": "normal"
  }
}
```

**Réponse** :
```json
{
  "analysis": {
    "id": "analysis_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "confidence_score": 0.94,
    "processing_time_ms": 2450,
    "model_used": "bert-large-multilingual",
    "results": {
      "summary": "Histoire fantastique mettant en scène un jeune prince dans une quête héroïque",
      "language_detected": "fr",
      "word_count": 1247,
      "reading_time_minutes": 5,
      "reading_level": "intermediate",
      "genre": {
        "primary": "fantasy",
        "secondary": "adventure",
        "confidence": 0.92
      },
      "themes": [
        {
          "theme": "courage",
          "confidence": 0.89,
          "evidence": ["prince courageux", "affronter le dragon"]
        },
        {
          "theme": "amitié",
          "confidence": 0.76,
          "evidence": ["compagnons fidèles", "entraide"]
        }
      ],
      "sentiment": {
        "overall": "positive",
        "score": 0.73,
        "emotions": {
          "joy": 0.45,
          "excitement": 0.62,
          "fear": 0.23,
          "sadness": 0.12
        }
      },
      "entities": [
        {
          "text": "prince",
          "type": "PERSON",
          "role": "protagonist",
          "confidence": 0.95
        },
        {
          "text": "royaume lointain",
          "type": "LOCATION",
          "role": "setting",
          "confidence": 0.88
        }
      ]
    }
  },
  "next_steps": [
    "scene_extraction",
    "character_analysis"
  ]
}
```

#### POST /api/v1/analysis/extract-scenes
**Description** : Extraction et structuration des scènes d'un contenu

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "content": "Il était une fois...",
  "scene_options": {
    "min_scene_length": 100,
    "max_scenes": 20,
    "include_transitions": true,
    "detect_dialogue": true
  },
  "style_preferences": {
    "visual_style": "realistic",
    "target_duration": "medium"
  }
}
```

**Réponse** :
```json
{
  "scene_extraction": {
    "id": "extraction_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "total_scenes": 12,
    "processing_time_ms": 3200,
    "scenes": [
      {
        "id": "scene_1",
        "sequence_number": 1,
        "title": "Le départ du prince",
        "content": "Le prince quitte le château royal au lever du soleil...",
        "duration_estimate": 45,
        "location": {
          "name": "Château royal",
          "type": "interior",
          "description": "Grande cour pavée avec tours imposantes"
        },
        "characters": [
          {
            "name": "Prince Alexandre",
            "role": "protagonist",
            "emotions": ["déterminé", "nostalgique"],
            "actions": ["monter à cheval", "regarder le château"]
          },
          {
            "name": "Roi père",
            "role": "supporting",
            "emotions": ["inquiet", "fier"],
            "actions": ["bénir son fils", "observer depuis la fenêtre"]
          }
        ],
        "mood": "solennel",
        "lighting": "lever de soleil doré",
        "camera_suggestions": [
          {
            "shot": "plan large",
            "description": "Vue d'ensemble du château au lever du soleil"
          },
          {
            "shot": "plan moyen",
            "description": "Le prince montant à cheval"
          }
        ],
        "dialogue": [
          {
            "character": "Roi père",
            "text": "Que les dieux te protègent, mon fils",
            "emotion": "solennel"
          }
        ],
        "transitions": {
          "from_previous": "fade_in",
          "to_next": "cut"
        }
      }
    ],
    "narrative_structure": {
      "act_1": [1, 2, 3, 4],
      "act_2": [5, 6, 7, 8, 9],
      "act_3": [10, 11, 12],
      "climax_scene": 10,
      "resolution_scene": 12
    }
  }
}
```

#### POST /api/v1/analysis/characters
**Description** : Analyse et caractérisation des personnages

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "content": "Il était une fois...",
  "character_options": {
    "include_minor_characters": true,
    "generate_descriptions": true,
    "analyze_relationships": true,
    "suggest_voice_types": true
  }
}
```

**Réponse** :
```json
{
  "character_analysis": {
    "id": "characters_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "total_characters": 8,
    "processing_time_ms": 1800,
    "characters": [
      {
        "id": "char_prince",
        "name": "Prince Alexandre",
        "role": "protagonist",
        "importance": "primary",
        "appearances": 15,
        "description": {
          "age": "20-25 ans",
          "gender": "masculin",
          "physical": "Grand, cheveux bruns, yeux verts, port noble",
          "personality": ["courageux", "déterminé", "loyal", "parfois impulsif"],
          "background": "Héritier du royaume, formé aux arts de la guerre",
          "motivation": "Sauver le royaume de la malédiction"
        },
        "character_arc": {
          "starting_point": "Prince inexpérimenté mais brave",
          "development": "Apprend la sagesse et l'humilité",
          "ending_point": "Roi sage et mature"
        },
        "relationships": [
          {
            "character_id": "char_mentor",
            "relationship": "mentor-student",
            "strength": 0.85
          },
          {
            "character_id": "char_princess",
            "relationship": "romantic",
            "strength": 0.92
          }
        ],
        "voice_suggestions": {
          "age_range": "young_adult",
          "tone": "noble",
          "accent": "neutral_french",
          "pitch": "medium"
        },
        "visual_references": [
          "Jeune homme noble médiéval",
          "Armure légère argentée",
          "Cape bleue royale"
        ]
      }
    ],
    "relationship_map": {
      "alliances": [
        ["char_prince", "char_mentor", "char_princess"]
      ],
      "conflicts": [
        ["char_prince", "char_villain"]
      ],
      "family_bonds": [
        ["char_prince", "char_king"]
      ]
    }
  }
}
```

### Image Generation

#### POST /api/v1/generation/images
**Description** : Génération d'images à partir de descriptions de scènes

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "scenes": [
    {
      "scene_id": "scene_1",
      "description": "Prince à cheval quittant un château au lever du soleil",
      "style": "realistic",
      "mood": "solennel",
      "lighting": "golden hour",
      "composition": "plan large"
    }
  ],
  "generation_options": {
    "resolution": "1920x1080",
    "quality": "high",
    "style_strength": 0.8,
    "seed": 42,
    "steps": 50,
    "guidance_scale": 7.5
  },
  "batch_size": 4
}
```

**Réponse** :
```json
{
  "image_generation": {
    "id": "generation_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "total_images": 4,
    "processing_time_ms": 45000,
    "gpu_time_ms": 42000,
    "model_used": "stable-diffusion-xl-1.0",
    "generated_images": [
      {
        "scene_id": "scene_1",
        "image_id": "img_001",
        "file_path": "generated/proj_123/scene_1_001.jpg",
        "url": "https://api.visiobook.com/api/v1/storage/download/img_001",
        "thumbnail_url": "https://api.visiobook.com/api/v1/storage/download/img_001_thumb",
        "resolution": "1920x1080",
        "file_size_bytes": 2048576,
        "generation_params": {
          "prompt": "A noble prince on horseback leaving a majestic castle at sunrise, golden light, cinematic composition",
          "negative_prompt": "blurry, low quality, distorted",
          "seed": 42,
          "steps": 50,
          "guidance_scale": 7.5
        },
        "quality_score": 0.89,
        "metadata": {
          "dominant_colors": ["#FFD700", "#8B4513", "#4169E1"],
          "composition_score": 0.92,
          "lighting_quality": 0.87
        }
      }
    ],
    "alternatives": [
      {
        "scene_id": "scene_1",
        "variations": 3,
        "best_candidate": "img_001"
      }
    ]
  }
}
```

#### GET /api/v1/generation/images/{generation_id}
**Description** : Statut et résultats d'une génération d'images

**Permissions** : user, premium, admin

**Réponse** :
```json
{
  "generation": {
    "id": "generation_123456789",
    "project_id": "proj_123456789",
    "status": "running",
    "progress_percent": 65,
    "current_scene": "scene_3",
    "estimated_completion": "2024-01-15T10:45:00Z",
    "completed_images": 8,
    "total_images": 12,
    "processing_stats": {
      "avg_time_per_image": 11250,
      "gpu_utilization": 87,
      "memory_usage": "18.5GB"
    }
  }
}
```

### Audio Generation

#### POST /api/v1/generation/audio/narration
**Description** : Génération de narration vocale

**Permissions** : user, premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "text": "Il était une fois, dans un royaume lointain...",
  "voice_options": {
    "language": "fr",
    "gender": "male",
    "age": "adult",
    "style": "narrative",
    "emotion": "neutral",
    "speed": 1.0,
    "pitch": 0.0
  },
  "audio_settings": {
    "format": "mp3",
    "quality": "high",
    "sample_rate": 44100,
    "normalize": true
  }
}
```

**Réponse** :
```json
{
  "audio_generation": {
    "id": "audio_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "audio_file": {
      "id": "audio_narration_001",
      "file_path": "generated/proj_123/narration.mp3",
      "url": "https://api.visiobook.com/api/v1/storage/download/audio_narration_001",
      "duration_seconds": 180,
      "file_size_bytes": 4320000,
      "format": "mp3",
      "sample_rate": 44100,
      "bitrate": 192
    },
    "processing_time_ms": 8500,
    "model_used": "tts-french-neural",
    "quality_metrics": {
      "clarity_score": 0.94,
      "naturalness_score": 0.88,
      "pronunciation_accuracy": 0.96
    }
  }
}
```

#### POST /api/v1/generation/audio/music
**Description** : Génération de musique d'ambiance

**Permissions** : premium, admin

**Requête** :
```json
{
  "project_id": "proj_123456789",
  "mood": "epic",
  "genre": "orchestral",
  "duration_seconds": 120,
  "instruments": ["strings", "brass", "percussion"],
  "tempo": "moderate",
  "key": "C_major",
  "style_references": ["medieval", "heroic"]
}
```

**Réponse** :
```json
{
  "music_generation": {
    "id": "music_123456789",
    "project_id": "proj_123456789",
    "status": "completed",
    "audio_file": {
      "id": "music_epic_001",
      "file_path": "generated/proj_123/background_music.mp3",
      "url": "https://api.visiobook.com/api/v1/storage/download/music_epic_001",
      "duration_seconds": 120,
      "file_size_bytes": 2880000,
      "format": "mp3",
      "loop_points": {
        "start": 10.5,
        "end": 110.2
      }
    },
    "processing_time_ms": 25000,
    "model_used": "musicgen-large",
    "composition_details": {
      "key": "C_major",
      "tempo_bpm": 80,
      "time_signature": "4/4",
      "dominant_instruments": ["strings", "brass"]
    }
  }
}
```

### Job Management

#### GET /api/v1/jobs
**Description** : Liste des tâches d'analyse en cours et terminées

**Permissions** : user, premium, admin

**Paramètres de requête** :
```
?project_id=proj_123456789
&status=pending,running,completed
&job_type=semantic,scene_extraction,image_generation
&page=1
&limit=20
```

**Réponse** :
```json
{
  "jobs": [
    {
      "id": "job_123456789",
      "project_id": "proj_123456789",
      "job_type": "image_generation",
      "status": "running",
      "priority": 5,
      "progress_percent": 75,
      "queue_position": 0,
      "estimated_completion": "2024-01-15T10:45:00Z",
      "resource_usage": {
        "gpu_time_ms": 180000,
        "cpu_time_ms": 45000,
        "memory_peak_mb": 8192
      },
      "created_at": "2024-01-15T10:30:00Z",
      "started_at": "2024-01-15T10:32:00Z"
    }
  ],
  "queue_stats": {
    "pending_jobs": 12,
    "running_jobs": 4,
    "available_workers": 2,
    "avg_wait_time_minutes": 8
  }
}
```

#### GET /api/v1/jobs/{job_id}
**Description** : Détails d'une tâche spécifique

**Permissions** : user, premium, admin

**Réponse** :
```json
{
  "job": {
    "id": "job_123456789",
    "project_id": "proj_123456789",
    "job_type": "semantic_analysis",
    "status": "completed",
    "priority": 5,
    "progress_percent": 100,
    "parameters": {
      "content_length": 1247,
      "language": "fr",
      "analysis_depth": "full"
    },
    "result": {
      "confidence_score": 0.94,
      "entities_found": 15,
      "themes_identified": 4,
      "processing_stages": [
        {
          "stage": "tokenization",
          "duration_ms": 150,
          "status": "completed"
        },
        {
          "stage": "embedding",
          "duration_ms": 800,
          "status": "completed"
        },
        {
          "stage": "classification",
          "duration_ms": 1200,
          "status": "completed"
        }
      ]
    },
    "resource_usage": {
      "gpu_time_ms": 2150,
      "cpu_time_ms": 450,
      "memory_peak_mb": 2048,
      "model_loading_time_ms": 300
    },
    "created_at": "2024-01-15T10:30:00Z",
    "started_at": "2024-01-15T10:30:15Z",
    "completed_at": "2024-01-15T10:32:30Z"
  }
}
```

#### DELETE /api/v1/jobs/{job_id}
**Description** : Annulation d'une tâche en attente

**Permissions** : user, premium, admin

**Réponse** :
```json
{
  "job_id": "job_123456789",
  "cancelled": true,
  "status": "cancelled",
  "message": "Job cancelled successfully"
}
```

### Model Management

#### GET /api/v1/models
**Description** : Liste des modèles IA disponibles

**Permissions** : admin

**Réponse** :
```json
{
  "models": [
    {
      "name": "bert-large-multilingual",
      "type": "semantic_analysis",
      "version": "1.0.0",
      "status": "loaded",
      "memory_usage_mb": 2048,
      "supported_languages": ["fr", "en", "es", "de"],
      "performance": {
        "avg_processing_time_ms": 1200,
        "accuracy_score": 0.94,
        "throughput_per_hour": 2400
      },
      "last_updated": "2024-01-10T10:00:00Z"
    },
    {
      "name": "stable-diffusion-xl",
      "type": "image_generation",
      "version": "1.0.0",
      "status": "loaded",
      "memory_usage_mb": 6800,
      "supported_resolutions": ["512x512", "1024x1024", "1920x1080"],
      "performance": {
        "avg_generation_time_ms": 12000,
        "quality_score": 0.89,
        "images_per_hour": 300
      },
      "last_updated": "2024-01-12T14:00:00Z"
    }
  ],
  "gpu_status": {
    "total_memory_gb": 24,
    "used_memory_gb": 18.5,
    "available_memory_gb": 5.5,
    "utilization_percent": 77
  }
}
```

## Flux de transactions CRUD

### Diagramme de séquence - Analyse sémantique

```mermaid
sequenceDiagram
    participant Client as Project Service
    participant API as AI Analysis API
    participant Queue as Celery Queue
    participant Worker as AI Worker
    participant GPU as GPU Models
    participant DB as Database

    Client->>API: POST /api/v1/analysis/semantic
    API->>Queue: Queue semantic analysis job
    Queue-->>API: Job queued
    API-->>Client: Return job ID

    Queue->>Worker: Assign job to worker
    Worker->>GPU: Load semantic model
    GPU-->>Worker: Model loaded
    Worker->>GPU: Process content analysis
    GPU-->>Worker: Analysis results
    Worker->>DB: Store analysis results
    Worker->>Queue: Mark job complete
    Queue->>Client: Notify completion (webhook)
```

### Diagramme de séquence - Génération d'images

```mermaid
sequenceDiagram
    participant Client as Project Service
    participant API as AI Analysis API
    participant Queue as Celery Queue
    participant Worker as GPU Worker
    participant Diffusion as Stable Diffusion
    participant Storage as Storage Service

    Client->>API: POST /api/v1/generation/images
    API->>Queue: Queue image generation job
    Queue-->>API: Job queued with priority
    API-->>Client: Return generation ID

    Queue->>Worker: Assign to GPU worker
    Worker->>Diffusion: Load diffusion model
    Diffusion-->>Worker: Model ready

    loop For each scene
        Worker->>Diffusion: Generate image
        Diffusion-->>Worker: Generated image
        Worker->>Storage: Store image file
        Storage-->>Worker: File stored
    end

    Worker->>Queue: Update progress
    Worker->>Queue: Mark job complete
```

### Diagramme de flux - Pipeline IA complet

```mermaid
flowchart TD
    Start([Content Input]) --> ValidateContent{Valid Content?}
    ValidateContent -->|No| ContentError[Return Content Error]
    ValidateContent -->|Yes| SemanticAnalysis[Semantic Analysis]

    SemanticAnalysis --> ExtractScenes[Scene Extraction]
    ExtractScenes --> AnalyzeCharacters[Character Analysis]
    AnalyzeCharacters --> CheckGPU{GPU Available?}

    CheckGPU -->|No| QueueImages[Queue Image Generation]
    CheckGPU -->|Yes| GenerateImages[Generate Images]

    QueueImages --> WaitGPU[Wait for GPU]
    WaitGPU --> GenerateImages

    GenerateImages --> GenerateAudio[Generate Audio]
    GenerateAudio --> AssembleAssets[Assemble All Assets]
    AssembleAssets --> StoreResults[Store in Database]
    StoreResults --> NotifyCompletion[Notify Project Service]
    NotifyCompletion --> Success[Analysis Complete]

    ContentError --> End([End])
    Success --> End
```

## Parcours utilisateurs détaillés

### Milestone 1: Importer un contenu

**US 1.1 - Import de fichiers**
```
1. POST /api/v1/analysis/semantic
   - Analyse automatique du contenu importé
   - Détection de la langue et du genre
   - Extraction des thèmes principaux

2. GET /api/v1/jobs/{job_id}
   - Suivi de la progression d'analyse
   - Récupération des résultats
```

**US 1.3 - Prévisualisation et résumé**
```
1. POST /api/v1/analysis/semantic
   - Génération automatique du résumé
   - Analyse de sentiment et d'émotion
   - Classification du niveau de lecture

2. POST /api/v1/analysis/extract-scenes
   - Extraction des scènes principales
   - Structuration narrative automatique
```

**US 1.4 - Validation du contenu**
```
1. POST /api/v1/analysis/characters
   - Identification des personnages principaux
   - Analyse des relations entre personnages
   - Suggestions de voix et d'apparence

2. GET /api/v1/jobs (multiple)
   - Vérification de la cohérence des analyses
   - Validation des résultats par l'utilisateur
```

**US 1.5 - Extraction des scènes clés**
```
1. POST /api/v1/analysis/extract-scenes
   - Segmentation intelligente du contenu
   - Identification des moments clés
   - Suggestions de transitions et de cadrages

2. POST /api/v1/generation/images
   - Génération des images de prévisualisation
   - Création des storyboards visuels
```

### Milestone 2: Personnaliser le style de l'animation

**US 2.1 - Choix du style graphique**
```
1. POST /api/v1/generation/images
   - Régénération avec nouveau style
   - Application des paramètres visuels
   - Optimisation selon les préférences

2. GET /api/v1/generation/images/{generation_id}
   - Suivi de la régénération
   - Comparaison des styles
```

**US 2.2 - Choix de la langue audio**
```
1. POST /api/v1/generation/audio/narration
   - Génération de la narration dans la langue choisie
   - Adaptation de la prosodie et de l'accent
   - Optimisation pour la compréhension

2. POST /api/v1/generation/audio/music
   - Génération de musique adaptée à la culture
   - Harmonisation avec le style narratif
```

### Milestone 3: Générer et visualiser une animation

**US 3.1 - Génération automatique**
```
1. POST /api/v1/generation/images (batch)
   - Génération de toutes les scènes
   - Application du style cohérent
   - Optimisation de la qualité

2. POST /api/v1/generation/audio/narration
   - Synthèse vocale complète
   - Synchronisation avec les images
   - Ajustement du timing

3. POST /api/v1/generation/audio/music
   - Création de la bande sonore
   - Adaptation aux émotions des scènes
   - Gestion des transitions musicales
```

**US 3.4 - Prévisualisation des scènes**
```
1. GET /api/v1/generation/images/{generation_id}
   - Récupération des images générées
   - Vérification de la qualité
   - Sélection des meilleures variantes

2. GET /api/v1/jobs (filtering by scene)
   - Suivi de la génération par scène
   - Identification des scènes problématiques
```

## Codes d'erreur

| Code | Message | Description |
|------|---------|-------------|
| 200 | Success | Opération réussie |
| 202 | Accepted | Tâche mise en queue pour traitement |
| 400 | Bad Request | Contenu invalide ou paramètres incorrects |
| 401 | Unauthorized | Token JWT manquant ou invalide |
| 403 | Forbidden | Permissions insuffisantes ou quota dépassé |
| 404 | Not Found | Tâche ou modèle non trouvé |
| 409 | Conflict | Tâche déjà en cours pour ce contenu |
| 413 | Payload Too Large | Contenu trop volumineux pour traitement |
| 422 | Unprocessable Entity | Contenu non analysable par l'IA |
| 429 | Too Many Requests | Limite de taux de traitement dépassée |
| 500 | Internal Server Error | Erreur interne du service IA |
| 503 | Service Unavailable | GPU indisponible ou modèle non chargé |
| 507 | Insufficient Storage | Espace insuffisant pour les résultats |

### Format d'erreur standardisé
```json
{
  "error": {
    "code": "GPU_MEMORY_EXHAUSTED",
    "message": "Insufficient GPU memory for image generation",
    "details": {
      "required_memory_gb": 12.5,
      "available_memory_gb": 8.2,
      "model_name": "stable-diffusion-xl",
      "suggested_action": "reduce_batch_size"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_123456789"
  }
}
```

### Codes d'erreur spécifiques
```json
{
  "VISIOBOOK_AI_PROCESSING_FAILED": "AI processing operation failed",
  "VISIOBOOK_GPU_UNAVAILABLE": "GPU resources temporarily unavailable",
  "VISIOBOOK_VALIDATION_FAILED": "Content validation failed",
  "VISIOBOOK_QUOTA_EXCEEDED": "AI processing quota exceeded",
  "VISIOBOOK_RESOURCE_NOT_FOUND": "Analysis job not found",
  "VISIOBOOK_INSUFFICIENT_PERMISSIONS": "User lacks required permissions",
  "VISIOBOOK_SERVICE_UNAVAILABLE": "AI service temporarily unavailable",
  "VISIOBOOK_RATE_LIMIT_EXCEEDED": "Rate limit exceeded for this endpoint"
}
```

## Versioning et migration

### Convention v1
- **URL Base** : `/api/v1/`
- **Headers** : `Accept: application/vnd.visiobook.v1+json`
- **Rétrocompatibilité** : Maintenue pendant 12 mois minimum

### Stratégie de migration
1. **Modèles IA** : Mise à jour progressive avec tests A/B
2. **Analyses existantes** : Retraitement automatique si nécessaire
3. **Formats de sortie** : Conversion automatique vers nouveaux formats
4. **GPU** : Migration transparente vers nouvelles architectures

### Exemple de migration v1 → v2
```json
{
  "migration": {
    "from_version": "v1",
    "to_version": "v2",
    "changes": [
      {
        "type": "model_upgraded",
        "component": "semantic_analysis",
        "old_model": "bert-large-multilingual",
        "new_model": "bert-large-multilingual-v2",
        "improvements": ["better_accuracy", "faster_processing"]
      },
      {
        "type": "feature_added",
        "component": "image_generation",
        "feature": "style_transfer",
        "description": "Advanced style transfer capabilities"
      }
    ],
    "breaking_changes": false,
    "migration_deadline": "2024-12-31"
  }
}
```

### Gestion des modèles IA
```json
{
  "model_lifecycle": {
    "semantic_analysis": {
      "current": "bert-large-multilingual-v1.0",
      "next": "bert-large-multilingual-v2.0",
      "migration_date": "2024-06-01",
      "compatibility": "backward_compatible"
    },
    "image_generation": {
      "current": "stable-diffusion-xl-1.0",
      "next": "stable-diffusion-xl-2.0",
      "migration_date": "2024-08-01",
      "compatibility": "requires_reprocessing"
    }
  },
  "performance_improvements": {
    "v2.0": {
      "semantic_analysis": "+25% speed, +15% accuracy",
      "image_generation": "+40% speed, +20% quality",
      "memory_usage": "-30% GPU memory"
    }
  }
}
```

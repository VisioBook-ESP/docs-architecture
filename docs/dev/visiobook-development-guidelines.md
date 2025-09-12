# Visiobook Development Guidelines

## Vue d'ensemble

Ce document définit les standards de développement obligatoires pour tous les microservices Visiobook. Ces guidelines garantissent la cohérence, la qualité et l'interopérabilité de l'ensemble de l'écosystème.

### Objectifs

- **Cohérence** : Standards uniformes pour tous les services
- **Qualité** : Pratiques de développement éprouvées
- **Interopérabilité** : Communication fluide entre microservices
- **Maintenabilité** : Code lisible et évolutif
- **Observabilité** : Monitoring et debugging efficaces

## 1. Standards de Développement Obligatoires

### 1.1 Structure de Projet Standardisée

#### Node.js + TypeScript (user-service, project-service, database-service)

```
microservice-name/
├── src/
│   ├── controllers/          # Contrôleurs API
│   ├── services/            # Logique métier
│   ├── models/              # Modèles de données
│   ├── middleware/          # Middlewares Express
│   ├── utils/               # Utilitaires
│   ├── config/              # Configuration
│   ├── types/               # Types TypeScript
│   └── app.ts               # Point d'entrée
├── prisma/
│   ├── schema.prisma        # Schéma base de données
│   └── migrations/          # Migrations SQL
├── tests/
│   ├── unit/                # Tests unitaires
│   ├── integration/         # Tests d'intégration
│   └── mocks/               # Mocks pour tests
├── docker/
│   ├── Dockerfile           # Image de production
│   └── Dockerfile.dev       # Image de développement
├── k8s/
│   ├── deployment.yaml      # Déploiement Kubernetes
│   ├── service.yaml         # Service Kubernetes
│   └── configmap.yaml       # Configuration
├── .env.example             # Variables d'environnement
├── package.json
├── tsconfig.json
├── jest.config.js
└── README.md
```

#### Python + FastAPI (ai-analysis-service, media-generation-service)

```
microservice-name/
├── app/
│   ├── api/
│   │   ├── v1/              # Endpoints API v1
│   │   └── dependencies.py  # Dépendances FastAPI
│   ├── core/
│   │   ├── config.py        # Configuration
│   │   └── security.py      # Sécurité
│   ├── models/              # Modèles Pydantic
│   ├── services/            # Logique métier
│   ├── utils/               # Utilitaires
│   └── main.py              # Point d'entrée
├── alembic/
│   ├── versions/            # Migrations base de données
│   └── env.py
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── docker/
│   ├── Dockerfile
│   └── Dockerfile.dev
├── k8s/
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
└── README.md
```

#### Go (storage-service, config-service)

```
microservice-name/
├── cmd/
│   └── server/
│       └── main.go          # Point d'entrée
├── internal/
│   ├── handlers/            # Handlers HTTP
│   ├── services/            # Logique métier
│   ├── models/              # Modèles de données
│   ├── middleware/          # Middlewares
│   ├── config/              # Configuration
│   └── utils/               # Utilitaires
├── pkg/                     # Packages réutilisables
├── migrations/              # Migrations SQL
├── tests/
├── docker/
├── k8s/
├── go.mod
├── go.sum
└── README.md
```

#### Vue.js + TypeScript (web-user-portal, admin-dashboard)

```
frontend-app/
├── src/
│   ├── components/          # Composants Vue
│   ├── views/               # Pages/Vues
│   ├── stores/              # Stores Pinia
│   ├── services/            # Services API
│   ├── types/               # Types TypeScript
│   ├── utils/               # Utilitaires
│   ├── assets/              # Assets statiques
│   └── main.ts              # Point d'entrée
├── public/
├── tests/
│   ├── unit/
│   └── e2e/
├── docker/
├── k8s/
├── package.json
├── vite.config.ts
├── tsconfig.json
└── README.md
```

### 1.2 Dockerfile Standardisé

#### Node.js Multi-Stage Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY tsconfig.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY src/ ./src/
COPY prisma/ ./prisma/

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:${PORT:-8080}/health || exit 1

USER nextjs

EXPOSE 8080

CMD ["node", "dist/app.js"]
```

#### Python FastAPI Dockerfile

```dockerfile
# Build stage
FROM python:3.11-slim AS builder

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt requirements-dev.txt ./

# Install Python dependencies
RUN pip install --no-cache-dir --user -r requirements.txt

# Production stage
FROM python:3.11-slim AS production

# Create non-root user
RUN useradd --create-home --shell /bin/bash app

WORKDIR /app

# Copy Python dependencies
COPY --from=builder /root/.local /home/app/.local

# Copy application
COPY --chown=app:app app/ ./app/
COPY --chown=app:app alembic/ ./alembic/
COPY --chown=app:app alembic.ini ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:${PORT:-8000}/health || exit 1

USER app

ENV PATH=/home/app/.local/bin:$PATH

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### Go Dockerfile

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main cmd/server/main.go

# Production stage
FROM alpine:latest AS production

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary
COPY --from=builder /app/main .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:${PORT:-8080}/health || exit 1

EXPOSE 8080

CMD ["./main"]
```

### 1.3 Configuration Helm Centralisée Sécurisée

#### Architecture de Configuration avec Contrôle d'Accès

L'architecture Visiobook utilise une approche **centralisée sécurisée** avec le repository `infra-helm-charts` pour garantir la gouvernance et le contrôle d'accès aux ressources critiques.

```yaml
Architecture Sécurisée:
  Repository Central: infra-helm-charts (Accès contrôlé)

  Développement Local (Minikube):
    - Accès: Développeurs complet
    - Ressources: CPU/RAM limitées, pas de GPU
    - Déploiement: helm install direct

  Staging:
    - Accès: Développeurs lecture seule via ArgoCD UI
    - Ressources: GPU limités, environnement contrôlé
    - Déploiement: ArgoCD automatique

  Production:
    - Accès: DevOps uniquement
    - Ressources: GPU production, secrets chiffrés
    - Déploiement: ArgoCD + approbation manuelle
```

#### Structure Repository Centralisé

```
github.com/VisioBook-ESP/infra-helm-charts/
├── charts/                         # Templates contrôlés
│   ├── visiobook-microservice/     # Chart générique backend
│   ├── visiobook-ai-service/       # Chart avec GPU (prod only)
│   └── visiobook-frontend/         # Chart frontend
├── environments/                   # Configuration par environnement
│   ├── dev/                       # Accès développeurs
│   │   ├── user-service.values.yaml
│   │   ├── project-service.values.yaml
│   │   └── secrets/               # Secrets dev non-sensibles
│   ├── staging/                   # Accès limité
│   │   ├── user-service.values.yaml
│   │   └── secrets/               # Secrets chiffrés
│   └── prod/                      # Accès DevOps uniquement
│       ├── user-service.values.yaml
│       └── secrets/               # Secrets Azure Key Vault
├── policies/                      # Politiques sécurité
│   ├── network-policies.yaml
│   ├── pod-security-policies.yaml
│   └── rbac.yaml
└── argocd/                        # Configuration ArgoCD
    ├── applications/
    └── projects/
```

#### Workflow de Déploiement Sécurisé

```yaml
1. Développement Local:
   Développeur:
     - Modifie code microservice
     - Test sur Minikube avec values-dev.yaml
     - Push code → CI/CD build image
   Ressources: CPU/RAM limitées, pas de GPU

2. Staging (Automatique):
   ArgoCD:
     - Détecte nouvelle image
     - Déploie automatiquement en staging
     - Tests d'intégration automatiques
   Accès Dev: Lecture seule via ArgoCD UI

3. Production (Manuel):
   DevOps:
     - Approuve le déploiement
     - ArgoCD déploie avec values-prod.yaml
     - Monitoring et rollback si nécessaire
   Accès Dev: Aucun accès direct
```

#### Exemple values.yaml par Environnement

```yaml
# environments/dev/user-service.values.yaml (Accès développeurs)
nameOverride: "user-service"

image:
  repository: ghcr.io/visiobook-esp/user-service
  tag: "latest"
  pullPolicy: IfNotPresent

deployment:
  replicaCount: 1  # Dev: 1 replica
  resources:
    requests:
      cpu: 50m
      memory: 64Mi
    limits:
      cpu: 200m      # Limité en dev
      memory: 256Mi   # Limité en dev
      # Pas de GPU en dev

configMap:
  SERVICE_NAME: "user-service"
  NODE_ENV: "development"
  LOG_LEVEL: "debug"
  API_GATEWAY_URL: "http://api-gateway-service:8080"

secrets:
  DATABASE_URL:
    secretName: "user-service-dev-secrets"
    key: "database-url"
  JWT_SECRET:
    secretName: "user-service-dev-secrets"
    key: "jwt-secret"

# environments/prod/ai-analysis-service.values.yaml (DevOps uniquement)
nameOverride: "ai-analysis-service"

deployment:
  replicaCount: 3  # Production: haute disponibilité
  resources:
    requests:
      cpu: 2000m
      memory: 8Gi
      nvidia.com/gpu: 1  # GPU production
    limits:
      cpu: 4000m
      memory: 16Gi
      nvidia.com/gpu: 1

configMap:
  SERVICE_NAME: "ai-analysis-service"
  PYTHON_ENV: "production"
  GPU_MEMORY_FRACTION: "0.9"  # Utilisation maximale GPU

secrets:
  HUGGINGFACE_TOKEN:
    secretName: "ai-analysis-prod-secrets"  # Azure Key Vault
    key: "hf-token"
  CUDA_VISIBLE_DEVICES:
    secretName: "ai-analysis-prod-secrets"
    key: "cuda-devices"
```

#### Contrôle d'Accès RBAC

```yaml
# policies/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
# Pas d'accès aux secrets ni aux GPU

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prod
  name: devops-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
# Accès complet pour DevOps en production
```

#### Workflow Développeur Sécurisé

```bash
# 1. Setup environnement de développement local
cd infra-helm-charts/environnement/dev
make setup  # Installe Minikube + Helm + ArgoCD + K9s

# 2. Déployer un microservice en développement (accès complet)
helm upgrade --install user-service \
  ../../charts/visiobook-microservice \
  -f environments/dev/user-service.values.yaml \
  -n dev --create-namespace

# 3. Appliquer les secrets dev (non-sensibles)
kubectl apply -f environments/dev/secrets/user-service-dev-secrets.yaml -n dev

# 4. Monitoring local
kubectl get pods -n dev
k9s  # Interface graphique

# 5. Tests locaux
kubectl port-forward service/user-service 8080:80 -n dev

# ❌ Pas d'accès direct staging/prod
# ✅ Monitoring via ArgoCD UI en lecture seule
```

#### Gestion des Secrets Sécurisée

```yaml
Développement (Minikube):
  - Secrets: Plain text dans repository (non-sensibles)
  - Ressources: CPU/RAM limitées
  - Accès: Développeurs complet
  - Rotation: Manuelle

Staging:
  - Secrets: Sealed Secrets chiffrés
  - Ressources: GPU limités pour tests
  - Accès: Développeurs lecture seule
  - Rotation: Automatique hebdomadaire

Production:
  - Secrets: Azure Key Vault + External Secrets Operator
  - Ressources: GPU production complets
  - Accès: DevOps uniquement
  - Rotation: Automatique quotidienne
  - Audit: Complet avec traçabilité
```

#### Avantages de l'Approche Centralisée Sécurisée

```yaml
Sécurité:
  ✅ Contrôle strict accès production
  ✅ GPU protégés des développeurs
  ✅ Secrets chiffrés par environnement
  ✅ Audit complet des déploiements

Gouvernance:
  ✅ Standards uniformes appliqués
  ✅ Politiques sécurité centralisées
  ✅ Validation avant production
  ✅ Rollback contrôlé

Vélocité:
  ✅ Développeurs autonomes en local
  ✅ Déploiement automatique staging
  ✅ Templates réutilisables
  ✅ CI/CD intégré
```

## 2. Gestion des Données et Architecture Database

### 2.1 Stratégie Multi-ORM et Database-Service

#### Philosophie de Gestion des Données

L'architecture Visiobook adopte une approche **hybride** pour la gestion des données :

1. **Phase de développement** : Chaque microservice utilise sa base de données locale avec l'ORM de son choix
2. **Phase de consolidation** : Les schémas sont migrés vers le `database-service` centralisé
3. **Phase de production** : Les microservices se connectent au `database-service` via des contrats d'interface standardisés

```yaml
Cycle de Vie des Données:
  Développement:
    - DB locale par microservice
    - ORM libre selon la stack choisie
    - Migrations locales pour prototypage rapide

  Consolidation:
    - Extraction des schémas vers database-service
    - Harmonisation des conventions de nommage
    - Validation des contraintes inter-services

  Production:
    - Connexion centralisée via database-service
    - APIs standardisées pour accès aux données
    - Monitoring et optimisation centralisés
```

#### ORMs Recommandés par Stack

```yaml
Node.js/TypeScript:
  Recommandé: Prisma 5.6+
  Alternatives: TypeORM 0.3+, Sequelize 6.35+
  Avantages Prisma: Type-safety, migrations auto, introspection
  Avantages TypeORM: Flexibilité, decorators, Active Record

Python/FastAPI:
  Recommandé: SQLAlchemy 2.0+
  Alternatives: Tortoise ORM 0.20+, Django ORM
  Avantages SQLAlchemy: Maturité, performance, flexibilité
  Avantages Tortoise: Async natif, simplicité

Go:
  Recommandé: GORM 1.25+
  Alternatives: sqlx 1.3+, Ent
  Avantages GORM: Simplicité, conventions, hooks
  Avantages sqlx: Performance, contrôle SQL direct
```

#### Standards Obligatoires (Indépendants de l'ORM)

```yaml
Conventions de Nommage:
  Tables: snake_case (users, user_profiles, project_files)
  Colonnes: snake_case (created_at, updated_at, user_id)
  Index: idx_table_column (idx_users_email, idx_projects_status)
  Contraintes: fk_table_column (fk_profiles_user_id)

Champs Obligatoires:
  id: UUID v4 (primary key)
  created_at: timestamp with timezone
  updated_at: timestamp with timezone
  version: integer (optimistic locking)

Types de Données Standardisés:
  ID: UUID (36 chars)
  Email: VARCHAR(255)
  Timestamps: TIMESTAMPTZ
  JSON: JSONB (PostgreSQL)
  Texte: TEXT (contenu long)
```

#### Exemple Prisma (Node.js) - Recommandé

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Modèle de base standardisé
model User {
  id        String   @id @default(uuid()) @db.Uuid
  email     String   @unique @db.VarChar(255)
  username  String   @unique @db.VarChar(50)
  password  String   @db.VarChar(255)
  role      UserRole @default(USER)

  // Relations
  profile   Profile?
  projects  Project[]

  // Champs obligatoires
  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz
  updatedAt DateTime @updatedAt @map("updated_at") @db.Timestamptz
  version   Int      @default(1)

  @@map("users")
  @@index([email])
  @@index([username])
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}

model Profile {
  id       String  @id @default(uuid()) @db.Uuid
  userId   String  @unique @map("user_id") @db.Uuid
  user     User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  firstName String? @map("first_name") @db.VarChar(100)
  lastName  String? @map("last_name") @db.VarChar(100)
  avatar    String? @db.VarChar(500)
  bio       String? @db.Text

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz
  updatedAt DateTime @updatedAt @map("updated_at") @db.Timestamptz
  version   Int      @default(1)

  @@map("user_profiles")
}
```

#### Exemple TypeORM (Node.js) - Alternative

```typescript
// src/entities/user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, OneToOne, OneToMany, Index } from 'typeorm';
import { Profile } from './profile.entity';
import { Project } from './project.entity';

export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  MODERATOR = 'moderator'
}

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'varchar', length: 255, unique: true })
  @Index()
  email: string;

  @Column({ type: 'varchar', length: 50, unique: true })
  @Index()
  username: string;

  @Column({ type: 'varchar', length: 255 })
  password: string;

  @Column({ type: 'enum', enum: UserRole, default: UserRole.USER })
  role: UserRole;

  // Relations
  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  profile: Profile;

  @OneToMany(() => Project, project => project.user)
  projects: Project[];

  // Champs obligatoires
  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at', type: 'timestamptz' })
  updatedAt: Date;

  @Column({ type: 'int', default: 1 })
  version: number;
}
```

#### Service Pattern Agnostique

```typescript
// src/services/user.service.ts
import { CreateUserDto, UpdateUserDto, UserResponseDto } from '../types/user.types';

// Interface commune indépendante de l'ORM
export interface IUserRepository {
  create(data: CreateUserDto): Promise<UserResponseDto>;
  findById(id: string): Promise<UserResponseDto | null>;
  findByEmail(email: string): Promise<UserResponseDto | null>;
  update(id: string, data: UpdateUserDto): Promise<UserResponseDto>;
  delete(id: string): Promise<void>;
  findMany(pagination: PaginationParams): Promise<{ users: UserResponseDto[], total: number }>;
}

// Implémentation Prisma
export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async create(data: CreateUserDto): Promise<UserResponseDto> {
    try {
      const user = await this.prisma.user.create({
        data: {
          ...data,
          profile: data.firstName || data.lastName ? {
            create: {
              firstName: data.firstName,
              lastName: data.lastName,
            },
          } : undefined,
        },
        include: { profile: true },
      });

      return this.mapToDto(user);
    } catch (error) {
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        if (error.code === 'P2002') {
          throw new Error('User already exists');
        }
      }
      throw error;
    }
  }

  async findById(id: string): Promise<UserResponseDto | null> {
    const user = await this.prisma.user.findUnique({
      where: { id },
      include: { profile: true, projects: true },
    });

    return user ? this.mapToDto(user) : null;
  }

  private mapToDto(user: any): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      username: user.username,
      role: user.role,
      profile: user.profile ? {
        id: user.profile.id,
        firstName: user.profile.firstName,
        lastName: user.profile.lastName,
        avatar: user.profile.avatar,
        bio: user.profile.bio,
      } : null,
      createdAt: user.createdAt.toISOString(),
      updatedAt: user.updatedAt.toISOString(),
    };
  }
}

// Implémentation TypeORM
export class TypeORMUserRepository implements IUserRepository {
  constructor(private userRepository: Repository<User>) {}

  async create(data: CreateUserDto): Promise<UserResponseDto> {
    try {
      const user = this.userRepository.create(data);

      if (data.firstName || data.lastName) {
        user.profile = new Profile();
        user.profile.firstName = data.firstName;
        user.profile.lastName = data.lastName;
      }

      const savedUser = await this.userRepository.save(user);
      return this.mapToDto(savedUser);
    } catch (error) {
      if (error.code === '23505') { // PostgreSQL unique violation
        throw new Error('User already exists');
      }
      throw error;
    }
  }

  // ... autres méthodes similaires
}

// Service utilisant l'interface
export class UserService {
  constructor(private userRepository: IUserRepository) {}

  async createUser(data: CreateUserDto): Promise<UserResponseDto> {
    // Validation métier
    await this.validateUserData(data);

    // Délégation au repository
    return this.userRepository.create(data);
  }

  async getUserById(id: string): Promise<UserResponseDto> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }

  private async validateUserData(data: CreateUserDto): Promise<void> {
    // Validation email unique
    const existingUser = await this.userRepository.findByEmail(data.email);
    if (existingUser) {
      throw new Error('Email already in use');
    }

    // Autres validations métier...
  }
}
```

### 2.2 Migrations Locales et Synchronisation

#### Stratégie de Migration

```typescript
// src/migrations/migration.service.ts
export class MigrationService {
  constructor(private prisma: PrismaClient) {}

  async runMigrations(): Promise<void> {
    try {
      // 1. Exécuter les migrations Prisma locales
      await this.prisma.$executeRaw`SELECT 1`;

      // 2. Notifier le database-service des changements
      await this.notifyDatabaseService();

      console.log('Migrations completed successfully');
    } catch (error) {
      console.error('Migration failed:', error);
      throw error;
    }
  }

  private async notifyDatabaseService(): Promise<void> {
    const migrationInfo = {
      service: process.env.SERVICE_NAME,
      version: process.env.SERVICE_VERSION,
      schema: await this.getSchemaDefinition(),
      timestamp: new Date().toISOString(),
    };

    // Envoyer au database-service pour synchronisation
    await fetch(`${process.env.DATABASE_SERVICE_URL}/api/v1/migrations/sync`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.SERVICE_TOKEN}`,
      },
      body: JSON.stringify(migrationInfo),
    });
  }

  private async getSchemaDefinition(): Promise<object> {
    // Extraire la définition du schéma pour synchronisation
    return this.prisma.$queryRaw`
      SELECT table_name, column_name, data_type
      FROM information_schema.columns
      WHERE table_schema = 'public'
    `;
  }
}
```

### 2.3 Typage Strict et Contrats d'Interface

#### Types TypeScript Obligatoires

```typescript
// src/types/user.types.ts
import { User, Profile } from '@prisma/client';

// DTOs pour API
export interface CreateUserDto {
  email: string;
  username: string;
  password: string;
  firstName?: string;
  lastName?: string;
}

export interface UpdateUserDto {
  email?: string;
  username?: string;
  firstName?: string;
  lastName?: string;
}

export interface UserResponseDto {
  id: string;
  email: string;
  username: string;
  role: string;
  profile?: ProfileResponseDto;
  createdAt: string;
  updatedAt: string;
}

export interface ProfileResponseDto {
  id: string;
  firstName?: string;
  lastName?: string;
  avatar?: string;
  bio?: string;
}

// Types avec relations
export type UserWithProfile = User & {
  profile: Profile | null;
};

// Validation schemas avec Zod
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email(),
  username: z.string().min(3).max(20),
  password: z.string().min(8),
  firstName: z.string().optional(),
  lastName: z.string().optional(),
});

export const UpdateUserSchema = CreateUserSchema.partial().omit({ password: true });
```

### 2.4 Gestion Multi-ORM et Adaptations par Stack

#### Équivalences ORM entre Technologies

```yaml
Mapping ORM par Stack:
  Node.js/TypeScript:
    ORM: Prisma 5.6+
    Avantages: Type-safety, migrations automatiques, introspection
    Inconvénients: Moins flexible pour requêtes complexes

  Python:
    ORM: SQLAlchemy 2.0+ (recommandé) ou Tortoise ORM
    Avantages: Flexibilité maximale, performance, maturité
    Inconvénients: Courbe d'apprentissage plus élevée

  Go:
    ORM: GORM 1.25+ (recommandé) ou sqlx
    Avantages: Performance native, simplicité
    Inconvénients: Moins de fonctionnalités avancées
```

#### SQLAlchemy - Configuration Standardisée (Services Python)

```python
# app/models/base.py
from sqlalchemy import Column, String, DateTime, Integer, func
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import UUID
import uuid

Base = declarative_base()

class BaseModel(Base):
    __abstract__ = True

    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    version = Column(Integer, default=1)

# app/models/user.py
from sqlalchemy import Column, String, Enum, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.dialects.postgresql import UUID
import enum

from .base import BaseModel

class UserRole(enum.Enum):
    USER = "user"
    ADMIN = "admin"
    MODERATOR = "moderator"

class User(BaseModel):
    __tablename__ = "users"

    email = Column(String, unique=True, nullable=False)
    username = Column(String, unique=True, nullable=False)
    password = Column(String, nullable=False)
    role = Column(Enum(UserRole), default=UserRole.USER)

    # Relations
    profile = relationship("Profile", back_populates="user", uselist=False)
    projects = relationship("Project", back_populates="user")

class Profile(BaseModel):
    __tablename__ = "profiles"

    user_id = Column(UUID(as_uuid=True), ForeignKey("users.id"), unique=True)
    first_name = Column(String, nullable=True)
    last_name = Column(String, nullable=True)
    avatar = Column(String, nullable=True)
    bio = Column(String, nullable=True)

    # Relations
    user = relationship("User", back_populates="profile")
```

#### Service Pattern avec SQLAlchemy

```python
# app/services/user_service.py
from sqlalchemy.orm import Session
from sqlalchemy.exc import IntegrityError
from typing import Optional, List
from fastapi import HTTPException

from app.models.user import User, Profile
from app.schemas.user import UserCreate, UserUpdate
from app.core.security import get_password_hash

class UserService:
    def __init__(self, db: Session):
        self.db = db

    async def create_user(self, user_data: UserCreate) -> User:
        try:
            # Hash password
            hashed_password = get_password_hash(user_data.password)

            # Create user
            db_user = User(
                email=user_data.email,
                username=user_data.username,
                password=hashed_password
            )

            self.db.add(db_user)
            self.db.flush()  # Get user ID without committing

            # Create profile if data provided
            if user_data.first_name or user_data.last_name:
                db_profile = Profile(
                    user_id=db_user.id,
                    first_name=user_data.first_name,
                    last_name=user_data.last_name
                )
                self.db.add(db_profile)

            self.db.commit()
            self.db.refresh(db_user)

            return db_user

        except IntegrityError as e:
            self.db.rollback()
            if "email" in str(e):
                raise HTTPException(status_code=400, detail="Email already exists")
            elif "username" in str(e):
                raise HTTPException(status_code=400, detail="Username already exists")
            raise HTTPException(status_code=400, detail="User creation failed")

    async def get_user_by_id(self, user_id: str) -> Optional[User]:
        return self.db.query(User).filter(User.id == user_id).first()

    async def update_user(self, user_id: str, user_data: UserUpdate) -> Optional[User]:
        db_user = await self.get_user_by_id(user_id)
        if not db_user:
            return None

        # Update user fields
        for field, value in user_data.dict(exclude_unset=True).items():
            if hasattr(db_user, field):
                setattr(db_user, field, value)

        # Increment version for optimistic locking
        db_user.version += 1

        self.db.commit()
        self.db.refresh(db_user)

        return db_user

    async def delete_user(self, user_id: str) -> bool:
        db_user = await self.get_user_by_id(user_id)
        if not db_user:
            return False

        self.db.delete(db_user)
        self.db.commit()

        return True
```

#### GORM - Configuration Standardisée (Services Go)

```go
// internal/models/base.go
package models

import (
    "time"
    "github.com/google/uuid"
    "gorm.io/gorm"
)

type BaseModel struct {
    ID        uuid.UUID      `gorm:"type:uuid;primary_key;default:gen_random_uuid()" json:"id"`
    CreatedAt time.Time      `gorm:"autoCreateTime" json:"created_at"`
    UpdatedAt time.Time      `gorm:"autoUpdateTime" json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`
    Version   int            `gorm:"default:1" json:"version"`
}

// internal/models/user.go
package models

type UserRole string

const (
    UserRoleUser      UserRole = "user"
    UserRoleAdmin     UserRole = "admin"
    UserRoleModerator UserRole = "moderator"
)

type User struct {
    BaseModel
    Email    string    `gorm:"uniqueIndex;not null" json:"email"`
    Username string    `gorm:"uniqueIndex;not null" json:"username"`
    Password string    `gorm:"not null" json:"-"`
    Role     UserRole  `gorm:"default:user" json:"role"`

    // Relations
    Profile  *Profile  `gorm:"foreignKey:UserID" json:"profile,omitempty"`
    Projects []Project `gorm:"foreignKey:UserID" json:"projects,omitempty"`
}

type Profile struct {
    BaseModel
    UserID    uuid.UUID `gorm:"type:uuid;uniqueIndex" json:"user_id"`
    FirstName *string   `json:"first_name,omitempty"`
    LastName  *string   `json:"last_name,omitempty"`
    Avatar    *string   `json:"avatar,omitempty"`
    Bio       *string   `json:"bio,omitempty"`

    // Relations
    User User `gorm:"constraint:OnDelete:CASCADE" json:"-"`
}
```

#### Service Pattern avec GORM

```go
// internal/services/user_service.go
package services

import (
    "errors"
    "github.com/google/uuid"
    "gorm.io/gorm"
    "golang.org/x/crypto/bcrypt"

    "app/internal/models"
    "app/internal/dto"
)

type UserService struct {
    db *gorm.DB
}

func NewUserService(db *gorm.DB) *UserService {
    return &UserService{db: db}
}

func (s *UserService) CreateUser(userData dto.CreateUserRequest) (*models.User, error) {
    // Hash password
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(userData.Password), bcrypt.DefaultCost)
    if err != nil {
        return nil, err
    }

    // Start transaction
    tx := s.db.Begin()
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
        }
    }()

    // Create user
    user := &models.User{
        Email:    userData.Email,
        Username: userData.Username,
        Password: string(hashedPassword),
        Role:     models.UserRoleUser,
    }

    if err := tx.Create(user).Error; err != nil {
        tx.Rollback()
        if errors.Is(err, gorm.ErrDuplicatedKey) {
            return nil, errors.New("user already exists")
        }
        return nil, err
    }

    // Create profile if data provided
    if userData.FirstName != nil || userData.LastName != nil {
        profile := &models.Profile{
            UserID:    user.ID,
            FirstName: userData.FirstName,
            LastName:  userData.LastName,
        }

        if err := tx.Create(profile).Error; err != nil {
            tx.Rollback()
            return nil, err
        }

        user.Profile = profile
    }

    tx.Commit()
    return user, nil
}

func (s *UserService) GetUserByID(userID uuid.UUID) (*models.User, error) {
    var user models.User
    err := s.db.Preload("Profile").First(&user, userID).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, err
    }
    return &user, nil
}

func (s *UserService) UpdateUser(userID uuid.UUID, userData dto.UpdateUserRequest) (*models.User, error) {
    var user models.User
    if err := s.db.First(&user, userID).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, err
    }

    // Update fields
    if userData.Email != nil {
        user.Email = *userData.Email
    }
    if userData.Username != nil {
        user.Username = *userData.Username
    }

    // Increment version for optimistic locking
    user.Version++

    if err := s.db.Save(&user).Error; err != nil {
        return nil, err
    }

    return &user, nil
}

func (s *UserService) DeleteUser(userID uuid.UUID) error {
    result := s.db.Delete(&models.User{}, userID)
    if result.Error != nil {
        return result.Error
    }
    if result.RowsAffected == 0 {
        return gorm.ErrRecordNotFound
    }
    return nil
}
```

#### Contrats d'Interface Universels

```yaml
# Schemas OpenAPI partagés entre toutes les stacks
UserContract:
  CreateUser:
    type: object
    required: [email, username, password]
    properties:
      email:
        type: string
        format: email
      username:
        type: string
        minLength: 3
        maxLength: 20
      password:
        type: string
        minLength: 8
      firstName:
        type: string
        nullable: true
      lastName:
        type: string
        nullable: true

  UserResponse:
    type: object
    properties:
      id:
        type: string
        format: uuid
      email:
        type: string
      username:
        type: string
      role:
        type: string
        enum: [user, admin, moderator]
      profile:
        $ref: '#/components/schemas/ProfileResponse'
      createdAt:
        type: string
        format: date-time
      updatedAt:
        type: string
        format: date-time

  ErrorResponse:
    type: object
    properties:
      error:
        type: object
        properties:
          code:
            type: integer
          message:
            type: string
          correlationId:
            type: string
          timestamp:
            type: number
```

#### Database Service Multi-ORM

```typescript
// src/services/database-multi-orm.service.ts
export interface DatabaseAdapter {
  getConnection(): Promise<any>;
  executeQuery(query: string, params?: any[]): Promise<any>;
  transaction<T>(callback: (tx: any) => Promise<T>): Promise<T>;
  migrate(): Promise<void>;
  getSchemaDefinition(): Promise<object>;
}

export class PrismaAdapter implements DatabaseAdapter {
  constructor(private prisma: PrismaClient) {}

  async getConnection() {
    return this.prisma;
  }

  async executeQuery(query: string, params?: any[]) {
    return this.prisma.$queryRawUnsafe(query, ...params);
  }

  async transaction<T>(callback: (tx: any) => Promise<T>): Promise<T> {
    return this.prisma.$transaction(callback);
  }

  async migrate() {
    // Prisma migrations are handled via CLI
    await this.prisma.$executeRaw`SELECT 1`;
  }

  async getSchemaDefinition() {
    return this.prisma.$queryRaw`
      SELECT table_name, column_name, data_type
      FROM information_schema.columns
      WHERE table_schema = 'public'
    `;
  }
}

export class SQLAlchemyAdapter implements DatabaseAdapter {
  constructor(private sessionFactory: () => Session) {}

  async getConnection() {
    return this.sessionFactory();
  }

  async executeQuery(query: string, params?: any[]) {
    const session = this.sessionFactory();
    try {
      return await session.execute(text(query), params);
    } finally {
      await session.close();
    }
  }

  async transaction<T>(callback: (tx: any) => Promise<T>): Promise<T> {
    const session = this.sessionFactory();
    const tx = session.begin();
    try {
      const result = await callback(session);
      await tx.commit();
      return result;
    } catch (error) {
      await tx.rollback();
      throw error;
    } finally {
      await session.close();
    }
  }

  async migrate() {
    // Alembic migrations handled separately
    const session = this.sessionFactory();
    try {
      await session.execute(text('SELECT 1'));
    } finally {
      await session.close();
    }
  }

  async getSchemaDefinition() {
    const session = this.sessionFactory();
    try {
      return await session.execute(text(`
        SELECT table_name, column_name, data_type
        FROM information_schema.columns
        WHERE table_schema = 'public'
      `));
    } finally {
      await session.close();
    }
  }
}

export class DatabaseServiceMultiORM {
  private adapters: Map<string, DatabaseAdapter> = new Map();

  registerAdapter(serviceName: string, adapter: DatabaseAdapter) {
    this.adapters.set(serviceName, adapter);
  }

  async syncMigrations(serviceName: string, migrationInfo: any) {
    const adapter = this.adapters.get(serviceName);
    if (!adapter) {
      throw new Error(`No adapter found for service: ${serviceName}`);
    }

    // Synchronize schema changes across all adapters
    const schema = await adapter.getSchemaDefinition();

    // Store migration info for tracking
    await this.storeMigrationInfo(serviceName, migrationInfo, schema);

    // Notify other services of schema changes if needed
    await this.notifySchemaChanges(serviceName, schema);
  }

  private async storeMigrationInfo(serviceName: string, migrationInfo: any, schema: any) {
    // Implementation for storing migration tracking
  }

  private async notifySchemaChanges(serviceName: string, schema: any) {
    // Implementation for notifying other services
  }
}
```

## 3. Convention de Nommage des Routes API

### 3.1 Format Standardisé

#### Structure des URLs

```
/api/v{version}/{service-name}/{resource}[/{id}][/{sub-resource}]

Exemples:
- GET    /api/v1/users                    # Liste des utilisateurs
- POST   /api/v1/users                    # Créer un utilisateur
- GET    /api/v1/users/{id}               # Obtenir un utilisateur
- PUT    /api/v1/users/{id}               # Mettre à jour un utilisateur
- DELETE /api/v1/users/{id}               # Supprimer un utilisateur
- GET    /api/v1/users/{id}/projects      # Projets d'un utilisateur
- POST   /api/v1/users/{id}/projects      # Créer un projet pour un utilisateur
```

#### Conventions de Nommage

```yaml
Services:
  - users (user-service)
  - projects (project-service)
  - storage (storage-service)
  - analysis (ai-analysis-service)
  - media (media-generation-service)

Resources:
  - Toujours au pluriel
  - Kebab-case pour les mots composés
  - Pas de verbes dans les noms de ressources

Actions:
  - GET: Récupération (liste ou détail)
  - POST: Création
  - PUT: Mise à jour complète
  - PATCH: Mise à jour partielle
  - DELETE: Suppression

Query Parameters:
  - page, limit: Pagination
  - sort: Tri (ex: sort=createdAt:desc)
  - filter: Filtrage (ex: filter[status]=active)
  - include: Relations à inclure
```

### 3.2 Implémentation avec NestJS

```typescript
// src/controllers/user.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  UseGuards,
  HttpStatus,
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { UserService } from '../services/user.service';
import { CreateUserDto, UpdateUserDto, UserResponseDto } from '../dto/user.dto';
import { JwtAuthGuard } from '../guards/jwt-auth.guard';
import { PaginationDto } from '../dto/pagination.dto';

@ApiTags('Users')
@Controller('api/v1/users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 201, description: 'User created successfully', type: UserResponseDto })
  @ApiResponse({ status: 400, description: 'Bad request' })
  async createUser(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    return this.userService.createUser(createUserDto);
  }

  @Get()
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'Users retrieved successfully' })
  async getUsers(@Query() paginationDto: PaginationDto) {
    return this.userService.getUsers(paginationDto);
  }

  @Get(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async getUserById(@Param('id') id: string): Promise<UserResponseDto> {
    return this.userService.getUserById(id);
  }

  @Put(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Update user' })
  @ApiResponse({ status: 200, description: 'User updated successfully', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async updateUser(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    return this.userService.updateUser(id, updateUserDto);
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Delete user' })
  @ApiResponse({ status: 204, description: 'User deleted successfully' })
  @ApiResponse({ status: 404, description: 'User not found' })
  async deleteUser(@Param('id') id: string): Promise<void> {
    return this.userService.deleteUser(id);
  }

  // Sous-ressources
  @Get(':id/projects')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get user projects' })
  async getUserProjects(@Param('id') id: string) {
    return this.userService.getUserProjects(id);
  }

  @Post(':id/projects')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Create project for user' })
  async createUserProject(@Param('id') id: string, @Body() projectData: any) {
    return this.userService.createUserProject(id, projectData);
  }
}
```

### 3.3 Documentation OpenAPI Automatique

```typescript
// src/swagger/user.swagger.ts
import { OpenAPIV3 } from 'openapi-types';

export const userPaths: OpenAPIV3.PathsObject = {
  '/api/v1/users': {
    get: {
      tags: ['Users'],
      summary: 'Get all users',
      parameters: [
        {
          name: 'page',
          in: 'query',
          schema: { type: 'integer', default: 1 },
        },
        {
          name: 'limit',
          in: 'query',
          schema: { type: 'integer', default: 10 },
        },
      ],
      responses: {
        '200': {
          description: 'List of users',
          content: {
            'application/json': {
              schema: {
                type: 'object',
                properties: {
                  data: {
                    type: 'array',
                    items: { $ref: '#/components/schemas/User' },
                  },
                  pagination: { $ref: '#/components/schemas/Pagination' },
                },
              },
            },
          },
        },
      },
    },
    post: {
      tags: ['Users'],
      summary: 'Create a new user',
      requestBody: {
        required: true,
        content: {
          'application/json': {
            schema: { $ref: '#/components/schemas/CreateUser' },
          },
        },
      },
      responses: {
        '201': {
          description: 'User created successfully',
          content: {
            'application/json': {
              schema: { $ref: '#/components/schemas/User' },
            },
          },
        },
      },
    },
  },
};
```

## 4. Stratégie de Tests et Mocking

### 4.1 Mocking des Dépendances Externes

#### Configuration Jest pour Node.js

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/types/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};
```

#### Mocks pour Services Externes

```typescript
// tests/mocks/external-services.mock.ts
export const mockDatabaseService = {
  getConnection: jest.fn().mockResolvedValue({
    query: jest.fn(),
    release: jest.fn(),
  }),
  executeQuery: jest.fn(),
  transaction: jest.fn(),
};

export const mockStorageService = {
  uploadFile: jest.fn().mockResolvedValue({
    url: 'https://cdn.example.com/file.jpg',
    key: 'file-key-123',
  }),
  deleteFile: jest.fn().mockResolvedValue(true),
  getFileUrl: jest.fn().mockReturnValue('https://cdn.example.com/file.jpg'),
};

export const mockAIAnalysisService = {
  analyzeContent: jest.fn().mockResolvedValue({
    scenes: [
      { id: 1, description: 'Scene 1', characters: ['John', 'Jane'] },
    ],
    summary: 'Content summary',
    sentiment: 'positive',
  }),
  extractEntities: jest.fn().mockResolvedValue(['entity1', 'entity2']),
};

// tests/setup.ts
import { mockDatabaseService, mockStorageService, mockAIAnalysisService } from './mocks/external-services.mock';

// Mock des modules externes
jest.mock('../src/services/database.service', () => ({
  DatabaseService: jest.fn().mockImplementation(() => mockDatabaseService),
}));

jest.mock('../src/services/storage.service', () => ({
  StorageService: jest.fn().mockImplementation(() => mockStorageService),
}));

jest.mock('../src/services/ai-analysis.service', () => ({
  AIAnalysisService: jest.fn().mockImplementation(() => mockAIAnalysisService),
}));
```

### 4.2 Tests Unitaires et d'Intégration

#### Tests Unitaires

```typescript
// tests/unit/services/user.service.test.ts
import { UserService } from '../../../src/services/user.service';
import { PrismaClient } from '@prisma/client';
import { mockDeep, DeepMockProxy } from 'jest-mock-extended';

describe('UserService', () => {
  let userService: UserService;
  let prismaMock: DeepMockProxy<PrismaClient>;

  beforeEach(() => {
    prismaMock = mockDeep<PrismaClient>();
    userService = new UserService(prismaMock);
  });

  describe('createUser', () => {
    it('should create a user successfully', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        username: 'testuser',
        password: 'hashedpassword',
        firstName: 'John',
        lastName: 'Doe',
      };

      const expectedUser = {
        id: 'user-123',
        ...userData,
        createdAt: new Date(),
        updatedAt: new Date(),
        version: 1,
        profile: {
          id: 'profile-123',
          firstName: 'John',
          lastName: 'Doe',
        },
      };

      prismaMock.user.create.mockResolvedValue(expectedUser);

      // Act
      const result = await userService.createUser(userData);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(prismaMock.user.create).toHaveBeenCalledWith({
        data: {
          ...userData,
          profile: {
            create: {
              firstName: userData.firstName,
              lastName: userData.lastName,
            },
          },
        },
        include: {
          profile: true,
        },
      });
    });

    it('should throw error when user already exists', async () => {
      // Arrange
      const userData = {
        email: 'existing@example.com',
        username: 'existinguser',
        password: 'hashedpassword',
      };

      prismaMock.user.create.mockRejectedValue({
        code: 'P2002',
        message: 'Unique constraint violation',
      });

      // Act & Assert
      await expect(userService.createUser(userData)).rejects.toThrow('User already exists');
    });
  });
});
```

#### Tests d'Intégration

```typescript
// tests/integration/user.integration.test.ts
import request from 'supertest';
import { app } from '../../src/app';
import { PrismaClient } from '@prisma/client';

describe('User Integration Tests', () => {
  let prisma: PrismaClient;

  beforeAll(async () => {
    prisma = new PrismaClient();
    await prisma.$connect();
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  beforeEach(async () => {
    // Nettoyer la base de données de test
    await prisma.user.deleteMany();
  });

  describe('POST /api/v1/users', () => {
    it('should create a new user', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        username: 'testuser',
        password: 'password123',
        firstName: 'John',
        lastName: 'Doe',
      };

      // Act
      const response = await request(app)
        .post('/api/v1/users')
        .send(userData)
        .expect(201);

      // Assert
      expect(response.body).toMatchObject({
        email: userData.email,
        username: userData.username,
        profile: {
          firstName: userData.firstName,
          lastName: userData.lastName,
        },
      });

      // Vérifier en base de données
      const userInDb = await prisma.user.findUnique({
        where: { email: userData.email },
        include: { profile: true },
      });

      expect(userInDb).toBeTruthy();
      expect(userInDb?.email).toBe(userData.email);
    });
  });
});
```

### 4.3 Contrats de Test entre Services

#### Définition des Contrats

```typescript
// tests/contracts/user-service.contract.ts
export interface UserServiceContract {
  // Endpoints obligatoires
  endpoints: {
    createUser: {
      method: 'POST';
      path: '/api/v1/users';
      request: CreateUserDto;
      response: UserResponseDto;
      status: 201;
    };
    getUser: {
      method: 'GET';
      path: '/api/v1/users/{id}';
      response: UserResponseDto;
      status: 200;
    };
    updateUser: {
      method: 'PUT';
      path: '/api/v1/users/{id}';
      request: UpdateUserDto;
      response: UserResponseDto;
      status: 200;
    };
    deleteUser: {
      method: 'DELETE';
      path: '/api/v1/users/{id}';
      status: 204;
    };
  };

  // Health checks obligatoires
  healthChecks: {
    health: {
      method: 'GET';
      path: '/health';
      response: HealthCheckResponse;
      status: 200;
    };
    ready: {
      method: 'GET';
      path: '/ready';
      response: ReadinessCheckResponse;
      status: 200;
    };
  };
}
```

#### Tests de Contrat

```typescript
// tests/contracts/user-service.contract.test.ts
import { UserServiceContract } from './user-service.contract';
import request from 'supertest';
import { app } from '../../src/app';

describe('User Service Contract Tests', () => {
  describe('Endpoints Contract', () => {
    it('should respect createUser contract', async () => {
      const requestData = {
        email: 'contract@example.com',
        username: 'contractuser',
        password: 'password123',
      };

      const response = await request(app)
        .post('/api/v1/users')
        .send(requestData)
        .expect(201);

      // Vérifier la structure de la réponse
      expect(response.body).toMatchObject({
        id: expect.any(String),
        email: requestData.email,
        username: requestData.username,
        role: expect.any(String),
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      });
    });
  });

  describe('Health Checks Contract', () => {
    it('should respect health check contract', async () => {
      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.body).toMatchObject({
        status: 'healthy',
        timestamp: expect.any(String),
        service: expect.any(String),
        version: expect.any(String),
      });
    });
  });
});
```

## 5. Monitoring et Observabilité

### 5.1 Health Checks Standardisés

#### Implémentation Health Checks

##### Node.js + TypeScript

```typescript
// src/middleware/health.middleware.ts
import { Request, Response } from 'express';
import { PrismaClient } from '@prisma/client';

export interface HealthCheckResponse {
  status: 'healthy' | 'unhealthy';
  timestamp: string;
  service: string;
  version: string;
  uptime: number;
  dependencies: {
    database: 'healthy' | 'unhealthy';
    redis: 'healthy' | 'unhealthy';
    external_services: Record<string, 'healthy' | 'unhealthy'>;
  };
}

export interface ReadinessCheckResponse {
  status: 'ready' | 'not_ready';
  timestamp: string;
  checks: {
    database_migrations: boolean;
    external_dependencies: boolean;
    configuration: boolean;
  };
}

export class HealthCheckService {
  constructor(
    private prisma: PrismaClient,
    private redis: any // Redis client
  ) {}

  async healthCheck(): Promise<HealthCheckResponse> {
    const startTime = Date.now();

    const [databaseHealth, redisHealth, externalServicesHealth] = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkExternalServices(),
    ]);

    return {
      status: this.determineOverallHealth([databaseHealth, redisHealth, externalServicesHealth]),
      timestamp: new Date().toISOString(),
      service: process.env.SERVICE_NAME || 'unknown',
      version: process.env.SERVICE_VERSION || '1.0.0',
      uptime: process.uptime(),
      dependencies: {
        database: databaseHealth.status === 'fulfilled' ? 'healthy' : 'unhealthy',
        redis: redisHealth.status === 'fulfilled' ? 'healthy' : 'unhealthy',
        external_services: externalServicesHealth.status === 'fulfilled'
          ? externalServicesHealth.value
          : {},
      },
    };
  }

  async readinessCheck(): Promise<ReadinessCheckResponse> {
    const [migrationsCheck, dependenciesCheck, configCheck] = await Promise.allSettled([
      this.checkMigrations(),
      this.checkExternalDependencies(),
      this.checkConfiguration(),
    ]);

    return {
      status: this.determineReadiness([migrationsCheck, dependenciesCheck, configCheck]),
      timestamp: new Date().toISOString(),
      checks: {
        database_migrations: migrationsCheck.status === 'fulfilled',
        external_dependencies: dependenciesCheck.status === 'fulfilled',
        configuration: configCheck.status === 'fulfilled',
      },
    };
  }

  private async checkDatabase(): Promise<void> {
    await this.prisma.$queryRaw`SELECT 1`;
  }

  private async checkRedis(): Promise<void> {
    await this.redis.ping();
  }

  private async checkExternalServices(): Promise<Record<string, 'healthy' | 'unhealthy'>> {
    const services = {
      'api-gateway': process.env.API_GATEWAY_URL,
      'storage-service': process.env.STORAGE_SERVICE_URL,
      'database-service': process.env.DATABASE_SERVICE_URL,
    };

    const results: Record<string, 'healthy' | 'unhealthy'> = {};

    for (const [serviceName, serviceUrl] of Object.entries(services)) {
      if (serviceUrl) {
        try {
          const response = await fetch(`${serviceUrl}/health`, {
            method: 'GET',
            timeout: 5000,
          });
          results[serviceName] = response.ok ? 'healthy' : 'unhealthy';
        } catch {
          results[serviceName] = 'unhealthy';
        }
      }
    }

    return results;
  }

  private async checkMigrations(): Promise<void> {
    // Vérifier que les migrations sont à jour
    const pendingMigrations = await this.prisma.$queryRaw`
      SELECT * FROM _prisma_migrations WHERE finished_at IS NULL
    `;

    if (Array.isArray(pendingMigrations) && pendingMigrations.length > 0) {
      throw new Error('Pending migrations found');
    }
  }

  private async checkExternalDependencies(): Promise<void> {
    // Vérifier les dépendances critiques
    const criticalServices = ['database-service', 'api-gateway'];

    for (const service of criticalServices) {
      const serviceUrl = process.env[`${service.toUpperCase().replace('-', '_')}_URL`];
      if (serviceUrl) {
        const response = await fetch(`${serviceUrl}/health`, { timeout: 3000 });
        if (!response.ok) {
          throw new Error(`Critical service ${service} is unhealthy`);
        }
      }
    }
  }

  private async checkConfiguration(): Promise<void> {
    // Vérifier les variables d'environnement critiques
    const requiredEnvVars = [
      'DATABASE_URL',
      'JWT_SECRET',
      'SERVICE_NAME',
      'SERVICE_VERSION',
    ];

    for (const envVar of requiredEnvVars) {
      if (!process.env[envVar]) {
        throw new Error(`Missing required environment variable: ${envVar}`);
      }
    }
  }

  private determineOverallHealth(results: PromiseSettledResult<any>[]): 'healthy' | 'unhealthy' {
    return results.every(result => result.status === 'fulfilled') ? 'healthy' : 'unhealthy';
  }

  private determineReadiness(results: PromiseSettledResult<any>[]): 'ready' | 'not_ready' {
    return results.every(result => result.status === 'fulfilled') ? 'ready' : 'not_ready';
  }
}

// Routes Health Check
export const createHealthRoutes = (healthService: HealthCheckService) => {
  const router = Router();

  router.get('/health', async (req: Request, res: Response) => {
    try {
      const health = await healthService.healthCheck();
      const statusCode = health.status === 'healthy' ? 200 : 503;
      res.status(statusCode).json(health);
    } catch (error) {
      res.status(503).json({
        status: 'unhealthy',
        timestamp: new Date().toISOString(),
        error: error.message,
      });
    }
  });

  router.get('/ready', async (req: Request, res: Response) => {
    try {
      const readiness = await healthService.readinessCheck();
      const statusCode = readiness.status === 'ready' ? 200 : 503;
      res.status(statusCode).json(readiness);
    } catch (error) {
      res.status(503).json({
        status: 'not_ready',
        timestamp: new Date().toISOString(),
        error: error.message,
      });
    }
  });

  router.get('/metrics', async (req: Request, res: Response) => {
    // Métriques Prometheus
    const metrics = await healthService.getMetrics();
    res.set('Content-Type', 'text/plain');
    res.send(metrics);
  });

  return router;
};
```
#### Implémentation Health Checks

##### Node.js + TypeScript

```typescript
// src/middleware/health.middleware.ts
import { Request, Response } from 'express';
import { PrismaClient } from '@prisma/client';

export interface HealthCheckResponse {
  status: 'healthy' | 'unhealthy';
  timestamp: string;
  service: string;
  version: string;
  uptime: number;
  dependencies: {
    database: 'healthy' | 'unhealthy';
    redis: 'healthy' | 'unhealthy';
    external_services: Record<string, 'healthy' | 'unhealthy'>;
  };
}

export interface ReadinessCheckResponse {
  status: 'ready' | 'not_ready';
  timestamp: string;
  checks: {
    database_migrations: boolean;
    external_dependencies: boolean;
    configuration: boolean;
  };
}

export class HealthCheckService {
  constructor(
    private prisma: PrismaClient,
    private redis: any // Redis client
  ) {}

  async healthCheck(): Promise<HealthCheckResponse> {
    const startTime = Date.now();

    const [databaseHealth, redisHealth, externalServicesHealth] = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkExternalServices(),
    ]);

    return {
      status: this.determineOverallHealth([databaseHealth, redisHealth, externalServicesHealth]),
      timestamp: new Date().toISOString(),
      service: process.env.SERVICE_NAME || 'unknown',
      version: process.env.SERVICE_VERSION || '1.0.0',
      uptime: process.uptime(),
      dependencies: {
        database: databaseHealth.status === 'fulfilled' ? 'healthy' : 'unhealthy',
        redis: redisHealth.status === 'fulfilled' ? 'healthy' : 'unhealthy',
        external_services: externalServicesHealth.status === 'fulfilled'
          ? externalServicesHealth.value
          : {},
      },
    };
  }

  async readinessCheck(): Promise<ReadinessCheckResponse> {
    const [migrationsCheck, dependenciesCheck, configCheck] = await Promise.allSettled([
      this.checkMigrations(),
      this.checkExternalDependencies(),
      this.checkConfiguration(),
    ]);

    return {
      status: this.determineReadiness([migrationsCheck, dependenciesCheck, configCheck]),
      timestamp: new Date().toISOString(),
      checks: {
        database_migrations: migrationsCheck.status === 'fulfilled',
        external_dependencies: dependenciesCheck.status === 'fulfilled',
        configuration: configCheck.status === 'fulfilled',
      },
    };
  }

  private async checkDatabase(): Promise<void> {
    await this.prisma.$queryRaw`SELECT 1`;
  }

  private async checkRedis(): Promise<void> {
    await this.redis.ping();
  }

  private async checkExternalServices(): Promise<Record<string, 'healthy' | 'unhealthy'>> {
    const services = {
      'api-gateway': process.env.API_GATEWAY_URL,
      'storage-service': process.env.STORAGE_SERVICE_URL,
      'database-service': process.env.DATABASE_SERVICE_URL,
    };

    const results: Record<string, 'healthy' | 'unhealthy'> = {};

    for (const [serviceName, serviceUrl] of Object.entries(services)) {
      if (serviceUrl) {
        try {
          const response = await fetch(`${serviceUrl}/health`, {
            method: 'GET',
            timeout: 5000,
          });
          results[serviceName] = response.ok ? 'healthy' : 'unhealthy';
        } catch {
          results[serviceName] = 'unhealthy';
        }
      }
    }

    return results;
  }

  private async checkMigrations(): Promise<void> {
    // Vérifier que les migrations sont à jour
    const pendingMigrations = await this.prisma.$queryRaw`
      SELECT * FROM _prisma_migrations WHERE finished_at IS NULL
    `;

    if (Array.isArray(pendingMigrations) && pendingMigrations.length > 0) {
      throw new Error('Pending migrations found');
    }
  }

  private async checkExternalDependencies(): Promise<void> {
    // Vérifier les dépendances critiques
    const criticalServices = ['database-service', 'api-gateway'];

    for (const service of criticalServices) {
      const serviceUrl = process.env[`${service.toUpperCase().replace('-', '_')}_URL`];
      if (serviceUrl) {
        const response = await fetch(`${serviceUrl}/health`, { timeout: 3000 });
        if (!response.ok) {
          throw new Error(`Critical service ${service} is unhealthy`);
        }
      }
    }
  }

  private async checkConfiguration(): Promise<void> {
    // Vérifier les variables d'environnement critiques
    const requiredEnvVars = [
      'DATABASE_URL',
      'JWT_SECRET',
      'SERVICE_NAME',
      'SERVICE_VERSION',
    ];

    for (const envVar of requiredEnvVars) {
      if (!process.env[envVar]) {
        throw new Error(`Missing required environment variable: ${envVar}`);
      }
    }
  }

  private determineOverallHealth(results: PromiseSettledResult<any>[]): 'healthy' | 'unhealthy' {
    return results.every(result => result.status === 'fulfilled') ? 'healthy' : 'unhealthy';
  }

  private determineReadiness(results: PromiseSettledResult<any>[]): 'ready' | 'not_ready' {
    return results.every(result => result.status === 'fulfilled') ? 'ready' : 'not_ready';
  }
}

// Routes Health Check
export const createHealthRoutes = (healthService: HealthCheckService) => {
  const router = Router();

  router.get('/health', async (req: Request, res: Response) => {
    try {
      const health = await healthService.healthCheck();
      const statusCode = health.status === 'healthy' ? 200 : 503;
      res.status(statusCode).json(health);
    } catch (error) {
      res.status(503).json({
        status: 'unhealthy',
        timestamp: new Date().toISOString(),
        error: error.message,
      });
    }
  });

  router.get('/ready', async (req: Request, res: Response) => {
    try {
      const readiness = await healthService.readinessCheck();
      const statusCode = readiness.status === 'ready' ? 200 : 503;
      res.status(statusCode).json(readiness);
    } catch (error) {
      res.status(503).json({
        status: 'not_ready',
        timestamp: new Date().toISOString(),
        error: error.message,
      });
    }
  });

  router.get('/metrics', async (req: Request, res: Response) => {
    // Métriques Prometheus
    const metrics = await healthService.getMetrics();
    res.set('Content-Type', 'text/plain');
    res.send(metrics);
  });

  return router;
};
```

##### Python + FastAPI

```python
# app/core/health.py
from fastapi import APIRouter, HTTPException
from fastapi.responses import PlainTextResponse
from sqlalchemy.orm import Session
from sqlalchemy import text
from typing import Dict, Any, List
import asyncio
import time
import os
import httpx
from datetime import datetime

from app.core.database import get_db
from app.core.redis import get_redis
from app.core.config import settings

class HealthCheckService:
    def __init__(self, db: Session, redis_client):
        self.db = db
        self.redis = redis_client
        self.start_time = time.time()

    async def health_check(self) -> Dict[str, Any]:
        """Perform comprehensive health check"""
        checks = await asyncio.gather(
            self._check_database(),
            self._check_redis(),
            self._check_external_services(),
            return_exceptions=True
        )

        database_health = "healthy" if not isinstance(checks[0], Exception) else "unhealthy"
        redis_health = "healthy" if not isinstance(checks[1], Exception) else "unhealthy"
        external_services = checks[2] if not isinstance(checks[2], Exception) else {}

        overall_status = "healthy" if all(
            not isinstance(check, Exception) for check in checks
        ) else "unhealthy"

        return {
            "status": overall_status,
            "timestamp": datetime.utcnow().isoformat(),
            "service": settings.SERVICE_NAME,
            "version": settings.SERVICE_VERSION,
            "uptime": time.time() - self.start_time,
            "dependencies": {
                "database": database_health,
                "redis": redis_health,
                "external_services": external_services
            }
        }

    async def readiness_check(self) -> Dict[str, Any]:
        """Perform readiness check"""
        checks = await asyncio.gather(
            self._check_migrations(),
            self._check_external_dependencies(),
            self._check_configuration(),
            return_exceptions=True
        )

        migrations_ok = not isinstance(checks[0], Exception)
        dependencies_ok = not isinstance(checks[1], Exception)
        config_ok = not isinstance(checks[2], Exception)

        overall_status = "ready" if all([migrations_ok, dependencies_ok, config_ok]) else "not_ready"

        return {
            "status": overall_status,
            "timestamp": datetime.utcnow().isoformat(),
            "checks": {
                "database_migrations": migrations_ok,
                "external_dependencies": dependencies_ok,
                "configuration": config_ok
            }
        }

    async def _check_database(self) -> None:
        """Check database connectivity"""
        try:
            result = self.db.execute(text("SELECT 1"))
            result.fetchone()
        except Exception as e:
            raise Exception(f"Database check failed: {str(e)}")

    async def _check_redis(self) -> None:
        """Check Redis connectivity"""
        try:
            await self.redis.ping()
        except Exception as e:
            raise Exception(f"Redis check failed: {str(e)}")

    async def _check_external_services(self) -> Dict[str, str]:
        """Check external services health"""
        services = {
            "api-gateway": settings.API_GATEWAY_URL,
            "storage-service": settings.STORAGE_SERVICE_URL,
            "database-service": settings.DATABASE_SERVICE_URL,
        }

        results = {}
        async with httpx.AsyncClient(timeout=5.0) as client:
            for service_name, service_url in services.items():
                if service_url:
                    try:
                        response = await client.get(f"{service_url}/health")
                        results[service_name] = "healthy" if response.status_code == 200 else "unhealthy"
                    except Exception:
                        results[service_name] = "unhealthy"

        return results

    async def _check_migrations(self) -> None:
        """Check if database migrations are up to date"""
        try:
            # Check Alembic migration status
            result = self.db.execute(text("""
                SELECT version_num FROM alembic_version
                ORDER BY version_num DESC LIMIT 1
            """))
            current_version = result.fetchone()

            if not current_version:
                raise Exception("No migration version found")

        except Exception as e:
            raise Exception(f"Migration check failed: {str(e)}")

    async def _check_external_dependencies(self) -> None:
        """Check critical external dependencies"""
        critical_services = ["database-service", "api-gateway"]

        async with httpx.AsyncClient(timeout=3.0) as client:
            for service in critical_services:
                service_url = getattr(settings, f"{service.upper().replace('-', '_')}_URL", None)
                if service_url:
                    try:
                        response = await client.get(f"{service_url}/health")
                        if response.status_code != 200:
                            raise Exception(f"Critical service {service} is unhealthy")
                    except Exception as e:
                        raise Exception(f"Critical service {service} check failed: {str(e)}")

    async def _check_configuration(self) -> None:
        """Check required environment variables"""
        required_vars = [
            "DATABASE_URL",
            "JWT_SECRET",
            "SERVICE_NAME",
            "SERVICE_VERSION"
        ]

        missing_vars = [var for var in required_vars if not os.getenv(var)]
        if missing_vars:
            raise Exception(f"Missing required environment variables: {', '.join(missing_vars)}")


# Health check routes
def create_health_router() -> APIRouter:
    router = APIRouter()

    @router.get("/health")
    async def health_check(db: Session = Depends(get_db)):
        """Health check endpoint"""
        try:
            redis_client = await get_redis()
            health_service = HealthCheckService(db, redis_client)
            health = await health_service.health_check()

            status_code = 200 if health["status"] == "healthy" else 503
            return JSONResponse(content=health, status_code=status_code)

        except Exception as e:
            return JSONResponse(
                content={
                    "status": "unhealthy",
                    "timestamp": datetime.utcnow().isoformat(),
                    "error": str(e)
                },
                status_code=503
            )

    @router.get("/ready")
    async def readiness_check(db: Session = Depends(get_db)):
        """Readiness check endpoint"""
        try:
            redis_client = await get_redis()
            health_service = HealthCheckService(db, redis_client)
            readiness = await health_service.readiness_check()

            status_code = 200 if readiness["status"] == "ready" else 503
            return JSONResponse(content=readiness, status_code=status_code)

        except Exception as e:
            return JSONResponse(
                content={
                    "status": "not_ready",
                    "timestamp": datetime.utcnow().isoformat(),
                    "error": str(e)
                },
                status_code=503
            )

    @router.get("/metrics", response_class=PlainTextResponse)
    async def metrics():
        """Prometheus metrics endpoint"""
        from app.core.metrics import metrics_service
        return await metrics_service.get_metrics()

    return router
```

##### Go + Gin

```go
// internal/health/health.go
package health

import (
    "context"
    "database/sql"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strconv"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/go-redis/redis/v8"
    "gorm.io/gorm"
)

type HealthCheckResponse struct {
    Status       string                            `json:"status"`
    Timestamp    string                            `json:"timestamp"`
    Service      string                            `json:"service"`
    Version      string                            `json:"version"`
    Uptime       float64                           `json:"uptime"`
    Dependencies map[string]interface{}            `json:"dependencies"`
}

type ReadinessCheckResponse struct {
    Status    string            `json:"status"`
    Timestamp string            `json:"timestamp"`
    Checks    map[string]bool   `json:"checks"`
}

type HealthCheckService struct {
    db          *gorm.DB
    redis       *redis.Client
    startTime   time.Time
}

func NewHealthCheckService(db *gorm.DB, redisClient *redis.Client) *HealthCheckService {
    return &HealthCheckService{
        db:        db,
        redis:     redisClient,
        startTime: time.Now(),
    }
}

func (h *HealthCheckService) HealthCheck(ctx context.Context) (*HealthCheckResponse, error) {
    // Run health checks concurrently
    type checkResult struct {
        name   string
        status string
        err    error
    }

    checks := make(chan checkResult, 3)

    // Database check
    go func() {
        err := h.checkDatabase(ctx)
        status := "healthy"
        if err != nil {
            status = "unhealthy"
        }
        checks <- checkResult{"database", status, err}
    }()

    // Redis check
    go func() {
        err := h.checkRedis(ctx)
        status := "healthy"
        if err != nil {
            status = "unhealthy"
        }
        checks <- checkResult{"redis", status, err}
    }()

    // External services check
    go func() {
        services, err := h.checkExternalServices(ctx)
        checks <- checkResult{"external_services", "healthy", err}
    }()

    // Collect results
    dependencies := make(map[string]interface{})
    var overallStatus string = "healthy"

    for i := 0; i < 3; i++ {
        result := <-checks
        if result.name == "external_services" {
            if result.err == nil {
                dependencies[result.name] = "healthy"
            } else {
                dependencies[result.name] = "unhealthy"
                overallStatus = "unhealthy"
            }
        } else {
            dependencies[result.name] = result.status
            if result.status == "unhealthy" {
                overallStatus = "unhealthy"
            }
        }
    }

    return &HealthCheckResponse{
        Status:       overallStatus,
        Timestamp:    time.Now().UTC().Format(time.RFC3339),
        Service:      os.Getenv("SERVICE_NAME"),
        Version:      os.Getenv("SERVICE_VERSION"),
        Uptime:       time.Since(h.startTime).Seconds(),
        Dependencies: dependencies,
    }, nil
}

func (h *HealthCheckService) ReadinessCheck(ctx context.Context) (*ReadinessCheckResponse, error) {
    type checkResult struct {
        name string
        ok   bool
    }

    checks := make(chan checkResult, 3)

    // Migration check
    go func() {
        err := h.checkMigrations(ctx)
        checks <- checkResult{"database_migrations", err == nil}
    }()

    // Dependencies check
    go func() {
        err := h.checkExternalDependencies(ctx)
        checks <- checkResult{"external_dependencies", err == nil}
    }()

    // Configuration check
    go func() {
        err := h.checkConfiguration()
        checks <- checkResult{"configuration", err == nil}
    }()

    // Collect results
    checksMap := make(map[string]bool)
    overallStatus := "ready"

    for i := 0; i < 3; i++ {
        result := <-checks
        checksMap[result.name] = result.ok
        if !result.ok {
            overallStatus = "not_ready"
        }
    }

    return &ReadinessCheckResponse{
        Status:    overallStatus,
        Timestamp: time.Now().UTC().Format(time.RFC3339),
        Checks:    checksMap,
    }, nil
}

func (h *HealthCheckService) checkDatabase(ctx context.Context) error {
    var result int
    err := h.db.WithContext(ctx).Raw("SELECT 1").Scan(&result).Error
    if err != nil {
        return fmt.Errorf("database check failed: %w", err)
    }
    return nil
}

func (h *HealthCheckService) checkRedis(ctx context.Context) error {
    _, err := h.redis.Ping(ctx).Result()
    if err != nil {
        return fmt.Errorf("redis check failed: %w", err)
    }
    return nil
}

func (h *HealthCheckService) checkExternalServices(ctx context.Context) (map[string]string, error) {
    services := map[string]string{
        "api-gateway":      os.Getenv("API_GATEWAY_URL"),
        "storage-service":  os.Getenv("STORAGE_SERVICE_URL"),
        "database-service": os.Getenv("DATABASE_SERVICE_URL"),
    }

    results := make(map[string]string)
    client := &http.Client{Timeout: 5 * time.Second}

    for serviceName, serviceURL := range services {
        if serviceURL != "" {
            req, err := http.NewRequestWithContext(ctx, "GET", serviceURL+"/health", nil)
            if err != nil {
                results[serviceName] = "unhealthy"
                continue
            }

            resp, err := client.Do(req)
            if err != nil || resp.StatusCode != http.StatusOK {
                results[serviceName] = "unhealthy"
            } else {
                results[serviceName] = "healthy"
            }

            if resp != nil {
                resp.Body.Close()
            }
        }
    }

    return results, nil
}

func (h *HealthCheckService) checkMigrations(ctx context.Context) error {
    // Check if there are any pending migrations
    // This is a simplified check - in practice, you'd check your migration system
    var count int64
    err := h.db.WithContext(ctx).Raw("SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public'").Scan(&count).Error
    if err != nil {
        return fmt.Errorf("migration check failed: %w", err)
    }

    if count == 0 {
        return fmt.Errorf("no tables found - migrations may not have run")
    }

    return nil
}

func (h *HealthCheckService) checkExternalDependencies(ctx context.Context) error {
    criticalServices := []string{"database-service", "api-gateway"}
    client := &http.Client{Timeout: 3 * time.Second}

    for _, service := range criticalServices {
        serviceURL := os.Getenv(strings.ToUpper(strings.ReplaceAll(service, "-", "_")) + "_URL")
        if serviceURL != "" {
            req, err := http.NewRequestWithContext(ctx, "GET", serviceURL+"/health", nil)
            if err != nil {
                return fmt.Errorf("critical service %s check failed: %w", service, err)
            }

            resp, err := client.Do(req)
            if err != nil {
                return fmt.Errorf("critical service %s is unreachable: %w", service, err)
            }

            if resp.StatusCode != http.StatusOK {
                return fmt.Errorf("critical service %s is unhealthy: status %d", service, resp.StatusCode)
            }

            resp.Body.Close()
        }
    }

    return nil
}

func (h *HealthCheckService) checkConfiguration() error {
    requiredVars := []string{
        "DATABASE_URL",
        "JWT_SECRET",
        "SERVICE_NAME",
        "SERVICE_VERSION",
    }

    var missingVars []string
    for _, envVar := range requiredVars {
        if os.Getenv(envVar) == "" {
            missingVars = append(missingVars, envVar)
        }
    }

    if len(missingVars) > 0 {
        return fmt.Errorf("missing required environment variables: %v", missingVars)
    }

    return nil
}

// HTTP Handlers
func SetupHealthRoutes(r *gin.Engine, healthService *HealthCheckService) {
    r.GET("/health", func(c *gin.Context) {
        health, err := healthService.HealthCheck(c.Request.Context())
        if err != nil {
            c.JSON(http.StatusServiceUnavailable, gin.H{
                "status":    "unhealthy",
                "timestamp": time.Now().UTC().Format(time.RFC3339),
                "error":     err.Error(),
            })
            return
        }

        statusCode := http.StatusOK
        if health.Status == "unhealthy" {
            statusCode = http.StatusServiceUnavailable
        }

        c.JSON(statusCode, health)
    })

    r.GET("/ready", func(c *gin.Context) {
        readiness, err := healthService.ReadinessCheck(c.Request.Context())
        if err != nil {
            c.JSON(http.StatusServiceUnavailable, gin.H{
                "status":    "not_ready",
                "timestamp": time.Now().UTC().Format(time.RFC3339),
                "error":     err.Error(),
            })
            return
        }

        statusCode := http.StatusOK
        if readiness.Status == "not_ready" {
            statusCode = http.StatusServiceUnavailable
        }

        c.JSON(statusCode, readiness)
    })

    r.GET("/metrics", func(c *gin.Context) {
        // Prometheus metrics endpoint
        // Implementation would depend on your metrics service
        c.Header("Content-Type", "text/plain")
        c.String(http.StatusOK, "# Metrics endpoint - implement with prometheus/client_golang")
    })
}
```

### 5.2 Logging Structuré

#### Node.js + TypeScript - Configuration Winston

```typescript
// src/utils/logger.ts
import winston from 'winston';
import { Request } from 'express';

export interface LogContext {
  correlationId?: string;
  userId?: string;
  service: string;
  version: string;
  environment: string;
}

class Logger {
  private logger: winston.Logger;

  constructor() {
    this.logger = winston.createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json(),
        winston.format.printf(({ timestamp, level, message, ...meta }) => {
          return JSON.stringify({
            timestamp,
            level,
            message,
            service: process.env.SERVICE_NAME,
            version: process.env.SERVICE_VERSION,
            environment: process.env.NODE_ENV,
            ...meta,
          });
        })
      ),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({
          filename: 'logs/error.log',
          level: 'error'
        }),
        new winston.transports.File({
          filename: 'logs/combined.log'
        }),
      ],
    });
  }

  info(message: string, context?: Partial<LogContext>, meta?: any) {
    this.logger.info(message, { ...context, ...meta });
  }

  error(message: string, error?: Error, context?: Partial<LogContext>, meta?: any) {
    this.logger.error(message, {
      error: error ? {
        message: error.message,
        stack: error.stack,
        name: error.name,
      } : undefined,
      ...context,
      ...meta,
    });
  }

  warn(message: string, context?: Partial<LogContext>, meta?: any) {
    this.logger.warn(message, { ...context, ...meta });
  }

  debug(message: string, context?: Partial<LogContext>, meta?: any) {
    this.logger.debug(message, { ...context, ...meta });
  }

  // Méthode pour logger les requêtes HTTP
  logRequest(req: Request, responseTime: number, statusCode: number) {
    this.info('HTTP Request', {
      correlationId: req.headers['x-correlation-id'] as string,
      userId: req.user?.id,
    }, {
      method: req.method,
      url: req.url,
      userAgent: req.headers['user-agent'],
      ip: req.ip,
      responseTime,
      statusCode,
    });
  }

  // Méthode pour logger les erreurs de base de données
  logDatabaseError(operation: string, error: Error, context?: any) {
    this.error(`Database operation failed: ${operation}`, error, {
      service: process.env.SERVICE_NAME,
    }, {
      operation,
      context,
    });
  }

  // Méthode pour logger les appels vers services externes
  logExternalServiceCall(serviceName: string, endpoint: string, responseTime: number, success: boolean) {
    this.info('External service call', {
      service: process.env.SERVICE_NAME,
    }, {
      externalService: serviceName,
      endpoint,
      responseTime,
      success,
    });
  }
}

export const logger = new Logger();

// Middleware pour correlation ID
export const correlationIdMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const correlationId = req.headers['x-correlation-id'] as string ||
                       req.headers['x-request-id'] as string ||
                       `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

  req.correlationId = correlationId;
  res.setHeader('x-correlation-id', correlationId);

  next();
};

// Middleware pour logger les requêtes
export const requestLoggerMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const startTime = Date.now();

  res.on('finish', () => {
    const responseTime = Date.now() - startTime;
    logger.logRequest(req, responseTime, res.statusCode);
  });

  next();
};
```

#### Python + FastAPI - Configuration Structlog

```python
# app/core/logging.py
import structlog
import logging
import sys
import json
import time
import uuid
from typing import Dict, Any, Optional
from datetime import datetime
from fastapi import Request

from app.core.config import settings

# Configuration structlog
def configure_logging():
    """Configure structured logging with structlog"""

    # Processeurs pour structlog
    processors = [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.StackInfoRenderer(),
        structlog.dev.set_exc_info,
        structlog.processors.TimeStamper(fmt="iso"),
        add_service_context,
        structlog.processors.JSONRenderer()
    ]

    # Configuration pour développement
    if settings.DEBUG:
        processors = [
            structlog.contextvars.merge_contextvars,
            structlog.processors.add_log_level,
            structlog.processors.StackInfoRenderer(),
            structlog.dev.set_exc_info,
            structlog.processors.TimeStamper(fmt="iso"),
            add_service_context,
            structlog.dev.ConsoleRenderer(colors=True)
        ]

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.make_filtering_bound_logger(
            getattr(logging, settings.LOG_LEVEL.upper())
        ),
        logger_factory=structlog.WriteLoggerFactory(),
        cache_logger_on_first_use=True,
    )

    # Configuration du logger standard Python
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=getattr(logging, settings.LOG_LEVEL.upper()),
    )

def add_service_context(logger, method_name: str, event_dict: Dict[str, Any]) -> Dict[str, Any]:
    """Ajouter le contexte du service à tous les logs"""
    event_dict.update({
        "service": settings.SERVICE_NAME,
        "version": settings.SERVICE_VERSION,
        "environment": settings.ENVIRONMENT,
    })
    return event_dict

class StructuredLogger:
    """Logger structuré pour l'application"""

    def __init__(self):
        self.logger = structlog.get_logger()

    def info(self, message: str, **kwargs):
        """Log info message"""
        self.logger.info(message, **kwargs)

    def error(self, message: str, error: Optional[Exception] = None, **kwargs):
        """Log error message"""
        if error:
            kwargs.update({
                "error": {
                    "message": str(error),
                    "type": error.__class__.__name__,
                    "traceback": str(error.__traceback__) if error.__traceback__ else None
                }
            })
        self.logger.error(message, **kwargs)

    def warning(self, message: str, **kwargs):
        """Log warning message"""
        self.logger.warning(message, **kwargs)

    def debug(self, message: str, **kwargs):
        """Log debug message"""
        self.logger.debug(message, **kwargs)

    def log_request(self, request: Request, response_time: float, status_code: int):
        """Log HTTP request"""
        correlation_id = getattr(request.state, "correlation_id", None)
        user_id = getattr(request.state, "user_id", None)

        self.info(
            "HTTP Request",
            method=request.method,
            url=str(request.url),
            user_agent=request.headers.get("user-agent"),
            ip=request.client.host if request.client else None,
            response_time=response_time,
            status_code=status_code,
            correlation_id=correlation_id,
            user_id=user_id,
        )

    def log_database_error(self, operation: str, error: Exception, context: Optional[Dict] = None):
        """Log database operation error"""
        self.error(
            f"Database operation failed: {operation}",
            error=error,
            operation=operation,
            context=context or {},
        )

    def log_external_service_call(
        self,
        service_name: str,
        endpoint: str,
        response_time: float,
        success: bool,
        status_code: Optional[int] = None
    ):
        """Log external service call"""
        self.info(
            "External service call",
            external_service=service_name,
            endpoint=endpoint,
            response_time=response_time,
            success=success,
            status_code=status_code,
        )

# Initialiser le logger
configure_logging()
logger = StructuredLogger()

# Middleware pour correlation ID et logging des requêtes
async def logging_middleware(request: Request, call_next):
    """Middleware pour logging des requêtes avec correlation ID"""

    # Générer ou récupérer correlation ID
    correlation_id = (
        request.headers.get("x-correlation-id") or
        request.headers.get("x-request-id") or
        str(uuid.uuid4())
    )

    # Stocker dans le state de la requête
    request.state.correlation_id = correlation_id

    # Ajouter au contexte structlog
    structlog.contextvars.clear_contextvars()
    structlog.contextvars.bind_contextvars(
        correlation_id=correlation_id,
        request_id=correlation_id
    )

    start_time = time.time()

    # Traiter la requête
    response = await call_next(request)

    # Calculer le temps de réponse
    response_time = (time.time() - start_time) * 1000  # en millisecondes

    # Ajouter correlation ID aux headers de réponse
    response.headers["x-correlation-id"] = correlation_id
    response.headers["x-process-time"] = str(response_time)

    # Logger la requête
    logger.log_request(request, response_time, response.status_code)

    return response

# Décorateur pour logger les fonctions
def log_function_call(func_name: Optional[str] = None):
    """Décorateur pour logger les appels de fonction"""
    def decorator(func):
        async def async_wrapper(*args, **kwargs):
            name = func_name or func.__name__
            start_time = time.time()

            try:
                logger.debug(f"Starting {name}", function=name, args=len(args), kwargs=list(kwargs.keys()))
                result = await func(*args, **kwargs)
                duration = (time.time() - start_time) * 1000
                logger.debug(f"Completed {name}", function=name, duration=duration, success=True)
                return result
            except Exception as e:
                duration = (time.time() - start_time) * 1000
                logger.error(f"Failed {name}", error=e, function=name, duration=duration, success=False)
                raise

        def sync_wrapper(*args, **kwargs):
            name = func_name or func.__name__
            start_time = time.time()

            try:
                logger.debug(f"Starting {name}", function=name, args=len(args), kwargs=list(kwargs.keys()))
                result = func(*args, **kwargs)
                duration = (time.time() - start_time) * 1000
                logger.debug(f"Completed {name}", function=name, duration=duration, success=True)
                return result
            except Exception as e:
                duration = (time.time() - start_time) * 1000
                logger.error(f"Failed {name}", error=e, function=name, duration=duration, success=False)
                raise

        # Retourner le bon wrapper selon le type de fonction
        import asyncio
        if asyncio.iscoroutinefunction(func):
            return async_wrapper
        else:
            return sync_wrapper

    return decorator
```

#### Go + Gin - Configuration Logrus

```go
// internal/logging/logger.go
package logging

import (
    "context"
    "fmt"
    "os"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"
    "github.com/sirupsen/logrus"
)

type LogContext struct {
    CorrelationID string `json:"correlation_id,omitempty"`
    UserID        string `json:"user_id,omitempty"`
    Service       string `json:"service"`
    Version       string `json:"version"`
    Environment   string `json:"environment"`
}

type StructuredLogger struct {
    logger *logrus.Logger
}

func NewStructuredLogger() *StructuredLogger {
    logger := logrus.New()

    // Configuration du format JSON
    logger.SetFormatter(&logrus.JSONFormatter{
        TimestampFormat: time.RFC3339,
        FieldMap: logrus.FieldMap{
            logrus.FieldKeyTime:  "timestamp",
            logrus.FieldKeyLevel: "level",
            logrus.FieldKeyMsg:   "message",
        },
    })

    // Configuration du niveau de log
    level, err := logrus.ParseLevel(getEnvOrDefault("LOG_LEVEL", "info"))
    if err != nil {
        level = logrus.InfoLevel
    }
    logger.SetLevel(level)

    // Configuration de la sortie
    logger.SetOutput(os.Stdout)

    return &StructuredLogger{logger: logger}
}

func (sl *StructuredLogger) getBaseFields() logrus.Fields {
    return logrus.Fields{
        "service":     getEnvOrDefault("SERVICE_NAME", "unknown"),
        "version":     getEnvOrDefault("SERVICE_VERSION", "1.0.0"),
        "environment": getEnvOrDefault("ENVIRONMENT", "development"),
    }
}

func (sl *StructuredLogger) Info(message string, fields logrus.Fields) {
    baseFields := sl.getBaseFields()
    for k, v := range fields {
        baseFields[k] = v
    }
    sl.logger.WithFields(baseFields).Info(message)
}

func (sl *StructuredLogger) Error(message string, err error, fields logrus.Fields) {
    baseFields := sl.getBaseFields()
    for k, v := range fields {
        baseFields[k] = v
    }

    if err != nil {
        baseFields["error"] = map[string]interface{}{
            "message": err.Error(),
            "type":    fmt.Sprintf("%T", err),
        }
    }

    sl.logger.WithFields(baseFields).Error(message)
}

func (sl *StructuredLogger) Warning(message string, fields logrus.Fields) {
    baseFields := sl.getBaseFields()
    for k, v := range fields {
        baseFields[k] = v
    }
    sl.logger.WithFields(baseFields).Warning(message)
}

func (sl *StructuredLogger) Debug(message string, fields logrus.Fields) {
    baseFields := sl.getBaseFields()
    for k, v := range fields {
        baseFields[k] = v
    }
    sl.logger.WithFields(baseFields).Debug(message)
}

func (sl *StructuredLogger) LogRequest(c *gin.Context, responseTime time.Duration, statusCode int) {
    correlationID := c.GetString("correlation_id")
    userID := c.GetString("user_id")

    fields := logrus.Fields{
        "method":         c.Request.Method,
        "url":            c.Request.URL.String(),
        "user_agent":     c.Request.UserAgent(),
        "ip":             c.ClientIP(),
        "response_time":  responseTime.Milliseconds(),
        "status_code":    statusCode,
        "correlation_id": correlationID,
    }

    if userID != "" {
        fields["user_id"] = userID
    }

    sl.Info("HTTP Request", fields)
}

func (sl *StructuredLogger) LogDatabaseError(operation string, err error, context map[string]interface{}) {
    fields := logrus.Fields{
        "operation": operation,
    }

    if context != nil {
        for k, v := range context {
            fields[k] = v
        }
    }

    sl.Error(fmt.Sprintf("Database operation failed: %s", operation), err, fields)
}

func (sl *StructuredLogger) LogExternalServiceCall(serviceName, endpoint string, responseTime time.Duration, success bool, statusCode int) {
    fields := logrus.Fields{
        "external_service": serviceName,
        "endpoint":         endpoint,
        "response_time":    responseTime.Milliseconds(),
        "success":          success,
        "status_code":      statusCode,
    }

    sl.Info("External service call", fields)
}

// Instance globale du logger
var Logger = NewStructuredLogger()

// Middleware pour correlation ID et logging des requêtes
func LoggingMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()

        // Générer ou récupérer correlation ID
        correlationID := c.GetHeader("x-correlation-id")
        if correlationID == "" {
            correlationID = c.GetHeader("x-request-id")
        }
        if correlationID == "" {
            correlationID = uuid.New().String()
        }

        // Stocker dans le contexte Gin
        c.Set("correlation_id", correlationID)

        // Ajouter aux headers de réponse
        c.Header("x-correlation-id", correlationID)

        // Traiter la requête
        c.Next()

        // Calculer le temps de réponse
        responseTime := time.Since(start)

        // Logger la requête
        Logger.LogRequest(c, responseTime, c.Writer.Status())
    }
}

// Middleware pour ajouter le correlation ID au contexte
func CorrelationIDMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        correlationID := c.GetString("correlation_id")
        if correlationID == "" {
            correlationID = uuid.New().String()
            c.Set("correlation_id", correlationID)
        }

        // Ajouter au contexte de la requête
        ctx := context.WithValue(c.Request.Context(), "correlation_id", correlationID)
        c.Request = c.Request.WithContext(ctx)

        c.Next()
    }
}

// Fonction utilitaire pour récupérer les variables d'environnement
func getEnvOrDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

// Fonction pour récupérer le correlation ID depuis le contexte
func GetCorrelationID(ctx context.Context) string {
    if correlationID, ok := ctx.Value("correlation_id").(string); ok {
        return correlationID
    }
    return ""
}

// Décorateur pour logger les fonctions (exemple d'utilisation)
func LogFunctionCall(funcName string, fn func() error) error {
    start := time.Now()

    Logger.Debug(fmt.Sprintf("Starting %s", funcName), logrus.Fields{
        "function": funcName,
    })

    err := fn()
    duration := time.Since(start)

    if err != nil {
        Logger.Error(fmt.Sprintf("Failed %s", funcName), err, logrus.Fields{
            "function": funcName,
            "duration": duration.Milliseconds(),
            "success":  false,
        })
        return err
    }

    Logger.Debug(fmt.Sprintf("Completed %s", funcName), logrus.Fields{
        "function": funcName,
        "duration": duration.Milliseconds(),
        "success":  true,
    })

    return nil
}
```

### 5.3 Métriques Prometheus

#### Configuration Métriques

```typescript
// src/utils/metrics.ts
import promClient from 'prom-client';

export class MetricsService {
  private register: promClient.Registry;

  // Métriques HTTP
  private httpRequestDuration: promClient.Histogram<string>;
  private httpRequestTotal: promClient.Counter<string>;
  private httpRequestErrors: promClient.Counter<string>;

  // Métriques Base de Données
  private dbConnectionsActive: promClient.Gauge<string>;
  private dbQueryDuration: promClient.Histogram<string>;
  private dbQueryErrors: promClient.Counter<string>;

  // Métriques Business
  private businessOperations: promClient.Counter<string>;
  private businessOperationDuration: promClient.Histogram<string>;

  constructor() {
    this.register = new promClient.Registry();

    // Métriques par défaut (CPU, mémoire, etc.)
    promClient.collectDefaultMetrics({ register: this.register });

    this.initializeMetrics();
  }

  private initializeMetrics() {
    // HTTP Metrics
    this.httpRequestDuration = new promClient.Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.1, 0.5, 1, 2, 5],
      registers: [this.register],
    });

    this.httpRequestTotal = new promClient.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });

    this.httpRequestErrors = new promClient.Counter({
      name: 'http_request_errors_total',
      help: 'Total number of HTTP request errors',
      labelNames: ['method', 'route', 'error_type'],
      registers: [this.register],
    });

    // Database Metrics
    this.dbConnectionsActive = new promClient.Gauge({
      name: 'db_connections_active',
      help: 'Number of active database connections',
      registers: [this.register],
    });

    this.dbQueryDuration = new promClient.Histogram({
      name: 'db_query_duration_seconds',
      help: 'Duration of database queries in seconds',
      labelNames: ['operation', 'table'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 2],
      registers: [this.register],
    });

    this.dbQueryErrors = new promClient.Counter({
      name: 'db_query_errors_total',
      help: 'Total number of database query errors',
      labelNames: ['operation', 'table', 'error_type'],
      registers: [this.register],
    });

    // Business Metrics
    this.businessOperations = new promClient.Counter({
      name: 'business_operations_total',
      help: 'Total number of business operations',
      labelNames: ['operation', 'status'],
      registers: [this.register],
    });

    this.businessOperationDuration = new promClient.Histogram({
      name: 'business_operation_duration_seconds',
      help: 'Duration of business operations in seconds',
      labelNames: ['operation'],
      buckets: [0.1, 0.5, 1, 5, 10, 30],
      registers: [this.register],
    });
  }

  // Méthodes pour enregistrer les métriques HTTP
  recordHttpRequest(method: string, route: string, statusCode: number, duration: number) {
    this.httpRequestDuration
      .labels(method, route, statusCode.toString())
      .observe(duration / 1000);

    this.httpRequestTotal
      .labels(method, route, statusCode.toString())
      .inc();
  }

  recordHttpError(method: string, route: string, errorType: string) {
    this.httpRequestErrors
      .labels(method, route, errorType)
      .inc();
  }

  // Méthodes pour enregistrer les métriques DB
  setActiveConnections(count: number) {
    this.dbConnectionsActive.set(count);
  }

  recordDbQuery(operation: string, table: string, duration: number) {
    this.dbQueryDuration
      .labels(operation, table)
      .observe(duration / 1000);
  }

  recordDbError(operation: string, table: string, errorType: string) {
    this.dbQueryErrors
      .labels(operation, table, errorType)
      .inc();
  }

  // Méthodes pour enregistrer les métriques business
  recordBusinessOperation(operation: string, status: 'success' | 'error', duration?: number) {
    this.businessOperations
      .labels(operation, status)
      .inc();

    if (duration !== undefined) {
      this.businessOperationDuration
        .labels(operation)
        .observe(duration / 1000);
    }
  }

  // Obtenir toutes les métriques
  async getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

export const metricsService = new MetricsService();

// Middleware pour collecter les métriques HTTP
export const metricsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const startTime = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - startTime;
    const route = req.route?.path || req.path;

    metricsService.recordHttpRequest(
      req.method,
      route,
      res.statusCode,
      duration
    );

    if (res.statusCode >= 400) {
      const errorType = res.statusCode >= 500 ? 'server_error' : 'client_error';
      metricsService.recordHttpError(req.method, route, errorType);
    }
  });

  next();
};
```

#### Python + FastAPI - Configuration Prometheus

```python
# app/core/metrics.py
from prometheus_client import Counter, Histogram, Gauge, CollectorRegistry, generate_latest
from prometheus_client.multiprocess import MultiProcessCollector
from fastapi import Request, Response
from typing import Dict, Any
import time
import os
import psutil

class MetricsService:
    """Service de métriques Prometheus pour FastAPI"""

    def __init__(self):
        self.registry = CollectorRegistry()

        # Métriques HTTP
        self.http_request_duration = Histogram(
            'http_request_duration_seconds',
            'Duration of HTTP requests in seconds',
            ['method', 'route', 'status_code'],
            buckets=[0.1, 0.5, 1, 2, 5],
            registry=self.registry
        )

        self.http_request_total = Counter(
            'http_requests_total',
            'Total number of HTTP requests',
            ['method', 'route', 'status_code'],
            registry=self.registry
        )

        self.http_request_errors = Counter(
            'http_request_errors_total',
            'Total number of HTTP request errors',
            ['method', 'route', 'error_type'],
            registry=self.registry
        )

        # Métriques Base de Données
        self.db_connections_active = Gauge(
            'db_connections_active',
            'Number of active database connections',
            registry=self.registry
        )

        self.db_query_duration = Histogram(
            'db_query_duration_seconds',
            'Duration of database queries in seconds',
            ['operation', 'table'],
            buckets=[0.01, 0.05, 0.1, 0.5, 1, 2],
            registry=self.registry
        )

        self.db_query_errors = Counter(
            'db_query_errors_total',
            'Total number of database query errors',
            ['operation', 'table', 'error_type'],
            registry=self.registry
        )

        # Métriques Business
        self.business_operations = Counter(
            'business_operations_total',
            'Total number of business operations',
            ['operation', 'status'],
            registry=self.registry
        )

        self.business_operation_duration = Histogram(
            'business_operation_duration_seconds',
            'Duration of business operations in seconds',
            ['operation'],
            buckets=[0.1, 0.5, 1, 5, 10, 30],
            registry=self.registry
        )

        # Métriques Système
        self.system_cpu_usage = Gauge(
            'system_cpu_usage_percent',
            'System CPU usage percentage',
            registry=self.registry
        )

        self.system_memory_usage = Gauge(
            'system_memory_usage_bytes',
            'System memory usage in bytes',
            registry=self.registry
        )

        # Démarrer la collecte des métriques système
        self._start_system_metrics_collection()

    def record_http_request(self, method: str, route: str, status_code: int, duration: float):
        """Enregistrer une requête HTTP"""
        self.http_request_duration.labels(
            method=method,
            route=route,
            status_code=str(status_code)
        ).observe(duration)

        self.http_request_total.labels(
            method=method,
            route=route,
            status_code=str(status_code)
        ).inc()

    def record_http_error(self, method: str, route: str, error_type: str):
        """Enregistrer une erreur HTTP"""
        self.http_request_errors.labels(
            method=method,
            route=route,
            error_type=error_type
        ).inc()

    def set_active_connections(self, count: int):
        """Définir le nombre de connexions actives"""
        self.db_connections_active.set(count)

    def record_db_query(self, operation: str, table: str, duration: float):
        """Enregistrer une requête base de données"""
        self.db_query_duration.labels(
            operation=operation,
            table=table
        ).observe(duration)

    def record_db_error(self, operation: str, table: str, error_type: str):
        """Enregistrer une erreur base de données"""
        self.db_query_errors.labels(
            operation=operation,
            table=table,
            error_type=error_type
        ).inc()

    def record_business_operation(self, operation: str, status: str, duration: float = None):
        """Enregistrer une opération business"""
        self.business_operations.labels(
            operation=operation,
            status=status
        ).inc()

        if duration is not None:
            self.business_operation_duration.labels(
                operation=operation
            ).observe(duration)

    def _start_system_metrics_collection(self):
        """Démarrer la collecte des métriques système"""
        import threading
        import time

        def collect_system_metrics():
            while True:
                try:
                    # CPU usage
                    cpu_percent = psutil.cpu_percent(interval=1)
                    self.system_cpu_usage.set(cpu_percent)

                    # Memory usage
                    memory = psutil.virtual_memory()
                    self.system_memory_usage.set(memory.used)

                    time.sleep(30)  # Collecter toutes les 30 secondes
                except Exception as e:
                    print(f"Error collecting system metrics: {e}")
                    time.sleep(60)

        thread = threading.Thread(target=collect_system_metrics, daemon=True)
        thread.start()

    async def get_metrics(self) -> str:
        """Obtenir toutes les métriques au format Prometheus"""
        return generate_latest(self.registry).decode('utf-8')


# Instance globale du service de métriques
metrics_service = MetricsService()


# Middleware pour collecter les métriques HTTP
async def metrics_middleware(request: Request, call_next):
    """Middleware pour collecter les métriques HTTP"""
    start_time = time.time()

    # Traiter la requête
    response = await call_next(request)

    # Calculer la durée
    duration = time.time() - start_time

    # Obtenir la route
    route = request.url.path
    if hasattr(request.state, 'route'):
        route = request.state.route

    # Enregistrer les métriques
    metrics_service.record_http_request(
        method=request.method,
        route=route,
        status_code=response.status_code,
        duration=duration
    )

    # Enregistrer les erreurs
    if response.status_code >= 400:
        error_type = "server_error" if response.status_code >= 500 else "client_error"
        metrics_service.record_http_error(
            method=request.method,
            route=route,
            error_type=error_type
        )

    return response


# Décorateur pour mesurer les opérations business
def measure_business_operation(operation_name: str):
    """Décorateur pour mesurer les opérations business"""
    def decorator(func):
        async def async_wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = await func(*args, **kwargs)
                duration = time.time() - start_time
                metrics_service.record_business_operation(operation_name, "success", duration)
                return result
            except Exception as e:
                duration = time.time() - start_time
                metrics_service.record_business_operation(operation_name, "error", duration)
                raise

        def sync_wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start_time
                metrics_service.record_business_operation(operation_name, "success", duration)
                return result
            except Exception as e:
                duration = time.time() - start_time
                metrics_service.record_business_operation(operation_name, "error", duration)
                raise

        import asyncio
        if asyncio.iscoroutinefunction(func):
            return async_wrapper
        else:
            return sync_wrapper

    return decorator


# Décorateur pour mesurer les requêtes base de données
def measure_db_operation(operation: str, table: str):
    """Décorateur pour mesurer les opérations base de données"""
    def decorator(func):
        async def async_wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = await func(*args, **kwargs)
                duration = time.time() - start_time
                metrics_service.record_db_query(operation, table, duration)
                return result
            except Exception as e:
                duration = time.time() - start_time
                error_type = type(e).__name__
                metrics_service.record_db_error(operation, table, error_type)
                raise

        def sync_wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start_time
                metrics_service.record_db_query(operation, table, duration)
                return result
            except Exception as e:
                duration = time.time() - start_time
                error_type = type(e).__name__
                metrics_service.record_db_error(operation, table, error_type)
                raise

        import asyncio
        if asyncio.iscoroutinefunction(func):
            return async_wrapper
        else:
            return sync_wrapper

    return decorator
```

#### Go + Gin - Configuration Prometheus

```go
// internal/metrics/metrics.go
package metrics

import (
    "fmt"
    "net/http"
    "runtime"
    "strconv"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "github.com/shirou/gopsutil/v3/cpu"
    "github.com/shirou/gopsutil/v3/mem"
)

type MetricsService struct {
    registry *prometheus.Registry

    // Métriques HTTP
    httpRequestDuration *prometheus.HistogramVec
    httpRequestTotal    *prometheus.CounterVec
    httpRequestErrors   *prometheus.CounterVec

    // Métriques Base de Données
    dbConnectionsActive *prometheus.GaugeVec
    dbQueryDuration     *prometheus.HistogramVec
    dbQueryErrors       *prometheus.CounterVec

    // Métriques Business
    businessOperations         *prometheus.CounterVec
    businessOperationDuration *prometheus.HistogramVec

    // Métriques Système
    systemCPUUsage    prometheus.Gauge
    systemMemoryUsage prometheus.Gauge
    goRoutines        prometheus.Gauge
}

func NewMetricsService() *MetricsService {
    registry := prometheus.NewRegistry()

    ms := &MetricsService{
        registry: registry,

        // HTTP Metrics
        httpRequestDuration: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Name:    "http_request_duration_seconds",
                Help:    "Duration of HTTP requests in seconds",
                Buckets: []float64{0.1, 0.5, 1, 2, 5},
            },
            []string{"method", "route", "status_code"},
        ),

        httpRequestTotal: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Name: "http_requests_total",
                Help: "Total number of HTTP requests",
            },
            []string{"method", "route", "status_code"},
        ),

        httpRequestErrors: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Name: "http_request_errors_total",
                Help: "Total number of HTTP request errors",
            },
            []string{"method", "route", "error_type"},
        ),

        // Database Metrics
        dbConnectionsActive: prometheus.NewGaugeVec(
            prometheus.GaugeOpts{
                Name: "db_connections_active",
                Help: "Number of active database connections",
            },
            []string{"database"},
        ),

        dbQueryDuration: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Name:    "db_query_duration_seconds",
                Help:    "Duration of database queries in seconds",
                Buckets: []float64{0.01, 0.05, 0.1, 0.5, 1, 2},
            },
            []string{"operation", "table"},
        ),

        dbQueryErrors: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Name: "db_query_errors_total",
                Help: "Total number of database query errors",
            },
            []string{"operation", "table", "error_type"},
        ),

        // Business Metrics
        businessOperations: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Name: "business_operations_total",
                Help: "Total number of business operations",
            },
            []string{"operation", "status"},
        ),

        businessOperationDuration: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Name:    "business_operation_duration_seconds",
                Help:    "Duration of business operations in seconds",
                Buckets: []float64{0.1, 0.5, 1, 5, 10, 30},
            },
            []string{"operation"},
        ),

        // System Metrics
        systemCPUUsage: prometheus.NewGauge(
            prometheus.GaugeOpts{
                Name: "system_cpu_usage_percent",
                Help: "System CPU usage percentage",
            },
        ),

        systemMemoryUsage: prometheus.NewGauge(
            prometheus.GaugeOpts{
                Name: "system_memory_usage_bytes",
                Help: "System memory usage in bytes",
            },
        ),

        goRoutines: prometheus.NewGauge(
            prometheus.GaugeOpts{
                Name: "go_goroutines",
                Help: "Number of goroutines",
            },
        ),
    }

    // Enregistrer toutes les métriques
    registry.MustRegister(
        ms.httpRequestDuration,
        ms.httpRequestTotal,
        ms.httpRequestErrors,
        ms.dbConnectionsActive,
        ms.dbQueryDuration,
        ms.dbQueryErrors,
        ms.businessOperations,
        ms.businessOperationDuration,
        ms.systemCPUUsage,
        ms.systemMemoryUsage,
        ms.goRoutines,
    )

    // Démarrer la collecte des métriques système
    ms.startSystemMetricsCollection()

    return ms
}

func (ms *MetricsService) RecordHTTPRequest(method, route string, statusCode int, duration time.Duration) {
    durationSeconds := duration.Seconds()
    statusCodeStr := strconv.Itoa(statusCode)

    ms.httpRequestDuration.WithLabelValues(method, route, statusCodeStr).Observe(durationSeconds)
    ms.httpRequestTotal.WithLabelValues(method, route, statusCodeStr).Inc()
}

func (ms *MetricsService) RecordHTTPError(method, route, errorType string) {
    ms.httpRequestErrors.WithLabelValues(method, route, errorType).Inc()
}

func (ms *MetricsService) SetActiveConnections(database string, count float64) {
    ms.dbConnectionsActive.WithLabelValues(database).Set(count)
}

func (ms *MetricsService) RecordDBQuery(operation, table string, duration time.Duration) {
    ms.dbQueryDuration.WithLabelValues(operation, table).Observe(duration.Seconds())
}

func (ms *MetricsService) RecordDBError(operation, table, errorType string) {
    ms.dbQueryErrors.WithLabelValues(operation, table, errorType).Inc()
}

func (ms *MetricsService) RecordBusinessOperation(operation, status string, duration *time.Duration) {
    ms.businessOperations.WithLabelValues(operation, status).Inc()

    if duration != nil {
        ms.businessOperationDuration.WithLabelValues(operation).Observe(duration.Seconds())
    }
}

func (ms *MetricsService) startSystemMetricsCollection() {
    go func() {
        ticker := time.NewTicker(30 * time.Second)
        defer ticker.Stop()

        for range ticker.C {
            // CPU usage
            if cpuPercent, err := cpu.Percent(time.Second, false); err == nil && len(cpuPercent) > 0 {
                ms.systemCPUUsage.Set(cpuPercent[0])
            }

            // Memory usage
            if memInfo, err := mem.VirtualMemory(); err == nil {
                ms.systemMemoryUsage.Set(float64(memInfo.Used))
            }

            // Goroutines
            ms.goRoutines.Set(float64(runtime.NumGoroutine()))
        }
    }()
}

func (ms *MetricsService) GetMetrics() (string, error) {
    gathering, err := ms.registry.Gather()
    if err != nil {
        return "", err
    }

    // Convert to Prometheus text format
    // Note: In a real implementation, you'd use prometheus.WriteToTextfile
    // or similar function to properly format the metrics
    return fmt.Sprintf("# Metrics collected at %s\n", time.Now().Format(time.RFC3339)), nil
}

// Instance globale du service de métriques
var GlobalMetricsService = NewMetricsService()

// Middleware pour collecter les métriques HTTP
func MetricsMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()

        // Traiter la requête
        c.Next()

        // Calculer la durée
        duration := time.Since(start)

        // Obtenir la route
        route := c.FullPath()
        if route == "" {
            route = c.Request.URL.Path
        }

        // Enregistrer les métriques
        GlobalMetricsService.RecordHTTPRequest(
            c.Request.Method,
            route,
            c.Writer.Status(),
            duration,
        )

        // Enregistrer les erreurs
        if c.Writer.Status() >= 400 {
            errorType := "client_error"
            if c.Writer.Status() >= 500 {
                errorType = "server_error"
            }
            GlobalMetricsService.RecordHTTPError(c.Request.Method, route, errorType)
        }
    }
}

// Décorateur pour mesurer les opérations business
func MeasureBusinessOperation(operation string, fn func() error) error {
    start := time.Now()

    err := fn()
    duration := time.Since(start)

    status := "success"
    if err != nil {
        status = "error"
    }

    GlobalMetricsService.RecordBusinessOperation(operation, status, &duration)

    return err
}

// Décorateur pour mesurer les opérations base de données
func MeasureDBOperation(operation, table string, fn func() error) error {
    start := time.Now()

    err := fn()
    duration := time.Since(start)

    if err != nil {
        errorType := fmt.Sprintf("%T", err)
        GlobalMetricsService.RecordDBError(operation, table, errorType)
    } else {
        GlobalMetricsService.RecordDBQuery(operation, table, duration)
    }

    return err
}

// Handler pour l'endpoint /metrics
func MetricsHandler() gin.HandlerFunc {
    handler := promhttp.HandlerFor(GlobalMetricsService.registry, promhttp.HandlerOpts{})

    return func(c *gin.Context) {
        handler.ServeHTTP(c.Writer, c.Request)
    }
}
```

### 5.4 KPIs Obligatoires par Service

#### KPIs Standardisés

```typescript
// src/utils/kpis.ts
export interface ServiceKPIs {
  // Performance KPIs
  averageResponseTime: number;
  p95ResponseTime: number;
  p99ResponseTime: number;
  throughput: number; // requests per second

  // Reliability KPIs
  uptime: number; // percentage
  errorRate: number; // percentage
  successRate: number; // percentage

  // Resource KPIs
  cpuUsage: number; // percentage
  memoryUsage: number; // percentage
  diskUsage: number; // percentage

  // Business KPIs (spécifiques par service)
  businessMetrics: Record<string, number>;
}

export class KPICollector {
  private metrics: Map<string, number[]> = new Map();
  private readonly windowSize = 300; // 5 minutes de données

  recordMetric(name: string, value: number) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }

    const values = this.metrics.get(name)!;
    values.push(value);

    // Garder seulement les dernières valeurs
    if (values.length > this.windowSize) {
      values.shift();
    }
  }

  getKPIs(): ServiceKPIs {
    return {
      averageResponseTime: this.getAverage('response_time'),
      p95ResponseTime: this.getPercentile('response_time', 95),
      p99ResponseTime: this.getPercentile('response_time', 99),
      throughput: this.getThroughput(),
      uptime: this.getUptime(),
      errorRate: this.getErrorRate(),
      successRate: this.getSuccessRate(),
      cpuUsage: this.getAverage('cpu_usage'),
      memoryUsage: this.getAverage('memory_usage'),
      diskUsage: this.getAverage('disk_usage'),
      businessMetrics: this.getBusinessMetrics(),
    };
  }

  private getAverage(metricName: string): number {
    const values = this.metrics.get(metricName) || [];
    if (values.length === 0) return 0;

    return values.reduce((sum, val) => sum + val, 0) / values.length;
  }

  private getPercentile(metricName: string, percentile: number): number {
    const values = this.metrics.get(metricName) || [];
    if (values.length === 0) return 0;

    const sorted = [...values].sort((a, b) => a - b);
    const index = Math.ceil((percentile / 100) * sorted.length) - 1;

    return sorted[index] || 0;
  }

  private getThroughput(): number {
    const requests = this.metrics.get('requests') || [];
    const timeWindow = Math.min(requests.length, this.windowSize);

    return timeWindow > 0 ? (requests.length / timeWindow) * 60 : 0; // per minute
  }

  private getUptime(): number {
    const healthChecks = this.metrics.get('health_checks') || [];
    if (healthChecks.length === 0) return 100;

    const successfulChecks = healthChecks.filter(check => check === 1).length;
    return (successfulChecks / healthChecks.length) * 100;
  }

  private getErrorRate(): number {
    const totalRequests = this.metrics.get('total_requests') || [];
    const errorRequests = this.metrics.get('error_requests') || [];

    const totalCount = totalRequests.reduce((sum, val) => sum + val, 0);
    const errorCount = errorRequests.reduce((sum, val) => sum + val, 0);

    return totalCount > 0 ? (errorCount / totalCount) * 100 : 0;
  }

  private getSuccessRate(): number {
    return 100 - this.getErrorRate();
  }

  private getBusinessMetrics(): Record<string, number> {
    const businessMetrics: Record<string, number> = {};

    // Métriques spécifiques par service
    const serviceName = process.env.SERVICE_NAME;

    switch (serviceName) {
      case 'user-service':
        businessMetrics.activeUsers = this.getAverage('active_users');
        businessMetrics.newRegistrations = this.getAverage('new_registrations');
        businessMetrics.loginAttempts = this.getAverage('login_attempts');
        break;

      case 'project-service':
        businessMetrics.projectsCreated = this.getAverage('projects_created');
        businessMetrics.projectsCompleted = this.getAverage('projects_completed');
        businessMetrics.averageProjectDuration = this.getAverage('project_duration');
        break;

      case 'ai-analysis-service':
        businessMetrics.analysisRequests = this.getAverage('analysis_requests');
        businessMetrics.averageAnalysisTime = this.getAverage('analysis_time');
        businessMetrics.gpuUtilization = this.getAverage('gpu_utilization');
        break;

      case 'storage-service':
        businessMetrics.filesUploaded = this.getAverage('files_uploaded');
        businessMetrics.storageUsed = this.getAverage('storage_used');
        businessMetrics.cdnHitRate = this.getAverage('cdn_hit_rate');
        break;
    }

    return businessMetrics;
  }
}

export const kpiCollector = new KPICollector();

// Middleware pour collecter les KPIs
export const kpiMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const startTime = Date.now();

  // Enregistrer la requête
  kpiCollector.recordMetric('requests', 1);
  kpiCollector.recordMetric('total_requests', 1);

  res.on('finish', () => {
    const responseTime = Date.now() - startTime;

    // Enregistrer le temps de réponse
    kpiCollector.recordMetric('response_time', responseTime);

    // Enregistrer les erreurs
    if (res.statusCode >= 400) {
      kpiCollector.recordMetric('error_requests', 1);
    }
  });

  next();
};
```

## 6. CI/CD et Infrastructure

### 6.1 Pipeline CI/CD Standardisé

#### GitHub Actions Workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run type checking
        run: npm run type-check

      - name: Run unit tests
        run: npm run test:unit
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Generate coverage report
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run security audit
        run: npm audit --audit-level high

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [test, security]
    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

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
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./docker/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.meta.outputs.tags }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'

    environment:
      name: staging
      url: https://staging.visiobook.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Helm
        uses: azure/setup-helm@v3
        with:
          version: '3.12.0'

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: '1.27.0'

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to staging
        run: |
          helm upgrade --install ${{ github.event.repository.name }} ./k8s/helm \
            --namespace staging \
            --create-namespace \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ github.sha }} \
            --set environment=staging \
            --wait --timeout=10m

      - name: Run smoke tests
        run: |
          kubectl wait --for=condition=ready pod -l app=${{ github.event.repository.name }} -n staging --timeout=300s
          npm run test:smoke -- --base-url=https://staging.visiobook.com

  deploy-production:
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    environment:
      name: production
      url: https://visiobook.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Helm
        uses: azure/setup-helm@v3
        with:
          version: '3.12.0'

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: '1.27.0'

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG_PRODUCTION }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to production (Blue-Green)
        run: |
          # Déploiement Blue-Green avec Helm
          CURRENT_VERSION=$(helm get values ${{ github.event.repository.name }} -n production -o json | jq -r '.image.tag // "none"')

          # Déployer la nouvelle version (Green)
          helm upgrade --install ${{ github.event.repository.name }}-green ./k8s/helm \
            --namespace production \
            --create-namespace \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ github.sha }} \
            --set environment=production \
            --set deployment.suffix=green \
            --wait --timeout=15m

      - name: Run production smoke tests
        run: |
          kubectl wait --for=condition=ready pod -l app=${{ github.event.repository.name }}-green -n production --timeout=300s
          npm run test:smoke -- --base-url=https://green.visiobook.com

      - name: Switch traffic to Green (Blue-Green)
        run: |
          # Basculer le trafic vers Green
          kubectl patch service ${{ github.event.repository.name }} -n production \
            -p '{"spec":{"selector":{"version":"green"}}}'

          # Attendre la stabilisation
          sleep 30

          # Tests de validation post-déploiement
          npm run test:smoke -- --base-url=https://visiobook.com

      - name: Cleanup old Blue deployment
        run: |
          # Supprimer l'ancienne version Blue après validation
          helm uninstall ${{ github.event.repository.name }}-blue -n production || true

#### Pipeline CI/CD pour Python + FastAPI

```yaml
# .github/workflows/python-ci-cd.yml
name: Python FastAPI CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  PYTHON_VERSION: "3.11"

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      - name: Run linting with flake8
        run: |
          flake8 app/ tests/ --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 app/ tests/ --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

      - name: Run type checking with mypy
        run: |
          mypy app/

      - name: Run security check with bandit
        run: |
          bandit -r app/ -f json -o bandit-report.json || true

      - name: Run unit tests
        run: |
          pytest tests/unit/ -v --cov=app --cov-report=xml --cov-report=html
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: |
          pytest tests/integration/ -v
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install safety bandit

      - name: Run safety check
        run: |
          safety check -r requirements.txt

      - name: Run bandit security scan
        run: |
          bandit -r app/ -f json -o bandit-report.json

  build:
    needs: [test, security]
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

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

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./docker/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

#### Pipeline CI/CD pour Go

```yaml
# .github/workflows/go-ci-cd.yml
name: Go CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  GO_VERSION: "1.21"

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: Download dependencies
        run: go mod download

      - name: Verify dependencies
        run: go mod verify

      - name: Run linting with golangci-lint
        uses: golangci/golangci-lint-action@v3
        with:
          version: latest
          args: --timeout=5m

      - name: Run security check with gosec
        uses: securecodewarrior/github-action-gosec@master
        with:
          args: './...'

      - name: Run unit tests
        run: |
          go test -v -race -coverprofile=coverage.out -covermode=atomic ./...
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: |
          go test -v -tags=integration ./tests/integration/...
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.out

  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: ${{ env.GO_VERSION }}

      - name: Run govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...

  build:
    needs: [test, security]
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

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

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./docker/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

### 6.2 Templates Helm Charts

#### Chart.yaml Standardisé

```yaml
# k8s/helm/Chart.yaml
apiVersion: v2
name: visiobook-microservice
description: Template Helm chart pour microservices Visiobook
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - visiobook
  - microservice
  - kubernetes
home: https://visiobook.com
sources:
  - https://github.com/visiobook/microservices
maintainers:
  - name: Visiobook Team
    email: devops@visiobook.com
```

#### Values.yaml Template

```yaml
# k8s/helm/values.yaml
# Configuration par défaut pour tous les microservices Visiobook

# Image configuration
image:
  repository: ghcr.io/visiobook/service-name
  tag: "latest"
  pullPolicy: IfNotPresent

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: 8080
  annotations: {}

# Deployment configuration
deployment:
  replicaCount: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

  # Resource limits
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi

  # Health checks
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    failureThreshold: 3

  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 3
    failureThreshold: 3

# Auto-scaling
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# Ingress configuration
ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: api.visiobook.com
      paths:
        - path: /api/v1/service-name
          pathType: Prefix
  tls:
    - secretName: visiobook-tls
      hosts:
        - api.visiobook.com

# Environment variables
env:
  NODE_ENV: production
  LOG_LEVEL: info
  PORT: "8080"

# Secrets (référencés depuis des secrets Kubernetes)
secrets:
  DATABASE_URL:
    secretName: database-credentials
    key: url
  JWT_SECRET:
    secretName: jwt-credentials
    key: secret
  REDIS_URL:
    secretName: redis-credentials
    key: url

# ConfigMap data
configMap:
  PROMETHEUS_PORT: "9090"
  CORRELATION_ID_HEADER: "x-correlation-id"
  API_GATEWAY_URL: "http://api-gateway-service:8080"

# Service Account
serviceAccount:
  create: true
  annotations: {}
  name: ""

# Pod Security Context
podSecurityContext:
  fsGroup: 1001
  runAsNonRoot: true
  runAsUser: 1001

# Security Context
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1001

# Node selector
nodeSelector: {}

# Tolerations
tolerations: []

# Affinity
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - visiobook-microservice
        topologyKey: kubernetes.io/hostname

# Monitoring
monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    interval: 30s
    path: /metrics
    port: metrics
```

#### Deployment Template

```yaml
# k8s/helm/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "visiobook-microservice.fullname" . }}
  labels:
    {{- include "visiobook-microservice.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.deployment.replicaCount }}
  {{- end }}
  strategy:
    {{- toYaml .Values.deployment.strategy | nindent 4 }}
  selector:
    matchLabels:
      {{- include "visiobook-microservice.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
      labels:
        {{- include "visiobook-microservice.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "visiobook-microservice.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
            - name: metrics
              containerPort: 9090
              protocol: TCP
          env:
            # Environment variables from values
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}

            # Secrets
            {{- range $key, $secret := .Values.secrets }}
            - name: {{ $key }}
              valueFrom:
                secretKeyRef:
                  name: {{ $secret.secretName }}
                  key: {{ $secret.key }}
            {{- end }}

          envFrom:
            - configMapRef:
                name: {{ include "visiobook-microservice.fullname" . }}

          livenessProbe:
            {{- toYaml .Values.deployment.livenessProbe | nindent 12 }}
          readinessProbe:
            {{- toYaml .Values.deployment.readinessProbe | nindent 12 }}
          resources:
            {{- toYaml .Values.deployment.resources | nindent 12 }}

          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: var-cache
              mountPath: /var/cache

      volumes:
        - name: tmp
          emptyDir: {}
        - name: var-cache
          emptyDir: {}

      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

## 7. Standards Multi-Stack et Gouvernance Technologique

### 7.1 Philosophie Multi-Stack

L'architecture Visiobook adopte une approche **technologie-agnostique** permettant aux équipes de choisir la stack la plus adaptée à leurs besoins, tout en maintenant la **cohérence architecturale** via des contrats d'interface standardisés.

#### Principes Fondamentaux

```yaml
Liberté Technologique:
  - Choix de stack par équipe/service
  - Expertise technique valorisée
  - Innovation encouragée
  - Performance optimisée

Cohérence Architecturale:
  - Contrats API standardisés
  - Monitoring uniforme
  - Sécurité homogène
  - CI/CD cohérent

Gouvernance:
  - Validation des nouvelles stacks
  - Support et formation
  - Documentation maintenue
  - Migration assistée
```

### 7.2 Stacks Technologiques Approuvées

#### Stack 1: Node.js + TypeScript (Services Backend Classiques)

```yaml
Cas d'Usage:
  - Services CRUD traditionnels
  - APIs REST/GraphQL
  - Intégrations tierces
  - Logique métier complexe

Technologies:
  Runtime: Node.js 18.18+ LTS
  Language: TypeScript 5.2+
  Framework: NestJS 10+ ou Express.js 4.18+
  ORM: Prisma 5.6+
  Validation: Zod 3.22+ ou class-validator
  Testing: Jest 29+ + Supertest
  Documentation: Swagger/OpenAPI 3.0+

Avantages:
  - Écosystème riche
  - Développement rapide
  - Typage strict
  - Communauté active

Services Recommandés:
  - user-service
  - project-service
  - notification-service
  - payment-service
```

#### Stack 2: Python + FastAPI (Services IA et Data)

```yaml
Cas d'Usage:
  - Machine Learning
  - Traitement de données
  - APIs haute performance
  - Calculs scientifiques

Technologies:
  Runtime: Python 3.11+
  Framework: FastAPI 0.104+
  ORM: SQLAlchemy 2.0+ ou Tortoise ORM
  ML Framework: PyTorch 2.1+ + Lightning
  Validation: Pydantic 2.4+
  Testing: pytest 7.4+ + pytest-asyncio
  Documentation: FastAPI auto-docs

Avantages:
  - Écosystème ML/IA
  - Performance async
  - Validation automatique
  - Documentation native

Services Recommandés:
  - ai-analysis-service
  - media-generation-service
  - content-ingestion-service
  - analytics-service
```

#### Stack 3: Go (Services Infrastructure et Performance)

```yaml
Cas d'Usage:
  - Services haute performance
  - Gateways et proxies
  - Outils système
  - Microservices légers

Technologies:
  Runtime: Go 1.21+
  Framework: Gin 1.9+ ou Fiber 2.50+
  ORM: GORM 1.25+ ou sqlx
  Testing: Testify + GoMock
  Documentation: Swagger avec go-swagger

Avantages:
  - Performance native
  - Concurrence native
  - Binaires légers
  - Déploiement simple

Services Recommandés:
  - api-gateway-service
  - storage-service
  - config-service
  - monitoring-service
```

#### Stack 4: Vue.js + TypeScript (Frontend)

```yaml
Cas d'Usage:
  - Applications web
  - Dashboards admin
  - Interfaces utilisateur
  - PWA

Technologies:
  Framework: Vue.js 3.3+ + TypeScript 5.0+
  Build Tool: Vite 4.0+
  State: Pinia + Vue Query (TanStack)
  UI: Vuetify 3+ + Material Design 3
  Testing: Vitest + Vue Testing Library

Avantages:
  - Courbe d'apprentissage douce
  - Performance optimisée
  - Écosystème mature
  - TypeScript natif

Services Recommandés:
  - web-user-portal
  - admin-dashboard
  - monitoring-dashboard
```

### 7.3 Matrice de Compatibilité

#### Équivalences entre Stacks

| Fonctionnalité | Node.js/TS | Python | Go | Vue.js |
|----------------|------------|--------|----|---------|
| **ORM** | Prisma | SQLAlchemy | GORM | N/A |
| **Validation** | Zod | Pydantic | validator | VeeValidate |
| **HTTP Client** | Axios | httpx | resty | Axios |
| **Testing** | Jest | pytest | Testify | Vitest |
| **Logging** | Winston | structlog | logrus | console |
| **Monitoring** | prom-client | prometheus-client | prometheus/client_golang | N/A |

#### Standards Transversaux Obligatoires

```yaml
API Design:
  - OpenAPI 3.0+ pour tous
  - Convention REST standardisée
  - Codes de statut HTTP uniformes
  - Headers de sécurité identiques

Monitoring:
  - Health checks: /health, /ready, /metrics
  - Métriques Prometheus format uniforme
  - Logging JSON structuré
  - Correlation IDs propagés

Sécurité:
  - JWT tokens standardisés
  - HTTPS obligatoire
  - Headers sécurité uniformes
  - Validation des entrées

Infrastructure:
  - Dockerfiles multi-stage
  - Helm charts adaptables
  - Variables d'environnement cohérentes
  - CI/CD pipelines similaires
```

### 7.4 Processus d'Approbation Nouvelles Stacks

#### Critères d'Évaluation

```yaml
Critères Techniques:
  - Maturité de l'écosystème
  - Performance et scalabilité
  - Sécurité et maintenance
  - Compatibilité infrastructure

Critères Organisationnels:
  - Expertise équipe disponible
  - Coût de formation
  - Support communautaire
  - Roadmap technologique

Critères Architecturaux:
  - Respect des contrats API
  - Intégration monitoring
  - Compatibilité CI/CD
  - Standards de sécurité
```

#### Processus de Validation

```yaml
Étapes:
  1. Proposition motivée par l'équipe
  2. Évaluation technique par l'architecture
  3. Proof of Concept sur service non-critique
  4. Validation des standards transversaux
  5. Formation équipe et documentation
  6. Approbation finale et adoption

Timeline:
  - Proposition: 1 semaine
  - Évaluation: 2 semaines
  - PoC: 2-4 semaines
  - Validation: 1 semaine
  - Formation: 1-2 semaines
```

### 7.2 Bases de Données Approuvées

#### Base de Données Principale

```yaml
PostgreSQL 15+:
  Usage: Données relationnelles principales
  Extensions: pgvector, pg_stat_statements
  Configuration:
    - Connection pooling (PgBouncer)
    - Read replicas pour lecture
    - Backup automatique quotidien
    - Monitoring avec pg_stat_monitor

Redis 7+:
  Usage: Cache, sessions, queues
  Configuration:
    - Cluster mode (3 masters + 3 replicas)
    - Persistence RDB + AOF
    - Memory policy: allkeys-lru
    - Monitoring avec RedisInsight

CosmosDB:
  Usage: Données NoSQL, documents
  Configuration:
    - Consistency level: Session
    - Auto-scaling 400-4000 RU/s
    - Multi-region replication
    - Partition strategy par tenant
```

#### Stockage de Fichiers

```yaml
Azure Blob Storage:
  Usage: Stockage principal fichiers
  Configuration:
    - Hot tier pour accès fréquent
    - Cool tier pour archivage
    - CDN Azure intégré
    - Lifecycle management

Azure CDN:
  Usage: Distribution globale
  Configuration:
    - Cache rules optimisées
    - Compression automatique
    - HTTPS obligatoire
    - Purge automatique
```

### 7.3 Outils de Développement Obligatoires

#### IDE et Extensions

```yaml
VS Code Extensions Obligatoires:
  - TypeScript + JavaScript
  - Prisma
  - ESLint + Prettier
  - GitLens
  - Docker
  - Kubernetes
  - Thunder Client (API testing)

Configuration Workspace:
  - Format on save activé
  - ESLint auto-fix activé
  - TypeScript strict mode
  - Prettier configuration unifiée
```

#### Outils CLI Requis

```bash
# Installation obligatoire pour tous les développeurs
npm install -g @nestjs/cli
npm install -g prisma
npm install -g typescript
npm install -g eslint
npm install -g prettier

# Outils Docker/Kubernetes
docker --version  # 24.0+
kubectl version   # 1.27+
helm version      # 3.12+

# Outils de test
npm install -g jest-cli
pip install pytest
```

## 8. Templates et Exemples de Code

### 8.1 Template Service Node.js + NestJS

#### Structure Complète

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';
import { logger } from './utils/logger';
import { correlationIdMiddleware } from './middleware/correlation-id.middleware';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global pipes
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));

  // Middleware
  app.use(correlationIdMiddleware);

  // Swagger documentation
  const config = new DocumentBuilder()
    .setTitle('Visiobook Microservice')
    .setDescription('API documentation')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document);

  // Start server
  const port = process.env.PORT || 8080;
  await app.listen(port);

  logger.info(`Service started on port ${port}`, {
    service: process.env.SERVICE_NAME,
    version: process.env.SERVICE_VERSION,
  });
}

bootstrap().catch(error => {
  logger.error('Failed to start service', error);
  process.exit(1);
});
```

#### Controller Template

```typescript
// src/controllers/user.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  HttpStatus,
  UseGuards,
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { UserService } from '../services/user.service';
import { CreateUserDto, UpdateUserDto, UserResponseDto } from '../dto/user.dto';
import { JwtAuthGuard } from '../guards/jwt-auth.guard';
import { PaginationDto } from '../dto/pagination.dto';

@ApiTags('Users')
@Controller('api/v1/users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 201, description: 'User created successfully', type: UserResponseDto })
  @ApiResponse({ status: 400, description: 'Bad request' })
  async createUser(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    return this.userService.createUser(createUserDto);
  }

  @Get()
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'Users retrieved successfully' })
  async getUsers(@Query() paginationDto: PaginationDto) {
    return this.userService.getUsers(paginationDto);
  }

  @Get(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async getUserById(@Param('id') id: string): Promise<UserResponseDto> {
    return this.userService.getUserById(id);
  }

  @Put(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Update user' })
  @ApiResponse({ status: 200, description: 'User updated successfully', type: UserResponseDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async updateUser(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    return this.userService.updateUser(id, updateUserDto);
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Delete user' })
  @ApiResponse({ status: 204, description: 'User deleted successfully' })
  @ApiResponse({ status: 404, description: 'User not found' })
  async deleteUser(@Param('id') id: string): Promise<void> {
    return this.userService.deleteUser(id);
  }
}
```

### 8.2 Template Service Python + FastAPI

#### Structure Complète

```python
# app/main.py
from fastapi import FastAPI, Request, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.responses import JSONResponse
import uvicorn
import time
import uuid
from contextlib import asynccontextmanager

from app.api.v1 import users, health
from app.core.config import settings
from app.core.logging import logger
from app.core.metrics import metrics_middleware


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    logger.info("Starting Visiobook microservice", extra={
        "service": settings.SERVICE_NAME,
        "version": settings.SERVICE_VERSION
    })
    yield
    # Shutdown
    logger.info("Shutting down Visiobook microservice")


app = FastAPI(
    title="Visiobook Microservice",
    description="API documentation for Visiobook microservice",
    version=settings.SERVICE_VERSION,
    lifespan=lifespan,
    docs_url="/api/docs",
    redoc_url="/api/redoc",
)

# Middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_HOSTS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=settings.ALLOWED_HOSTS
)


@app.middleware("http")
async def correlation_id_middleware(request: Request, call_next):
    correlation_id = (
        request.headers.get("x-correlation-id") or
        request.headers.get("x-request-id") or
        str(uuid.uuid4())
    )

    request.state.correlation_id = correlation_id

    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time

    response.headers["x-correlation-id"] = correlation_id
    response.headers["x-process-time"] = str(process_time)

    logger.info("HTTP Request", extra={
        "method": request.method,
        "url": str(request.url),
        "status_code": response.status_code,
        "process_time": process_time,
        "correlation_id": correlation_id,
    })

    return response


@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.status_code,
                "message": exc.detail,
                "correlation_id": getattr(request.state, "correlation_id", None),
                "timestamp": time.time(),
            }
        },
    )


# Include routers
app.include_router(health.router, tags=["Health"])
app.include_router(users.router, prefix="/api/v1", tags=["Users"])

if __name__ == "__main__":
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=settings.PORT,
        reload=settings.DEBUG,
        log_config=None,  # Use our custom logging
    )
```

#### API Router Template

```python
# app/api/v1/users.py
from fastapi import APIRouter, Depends, HTTPException, Query, status
from fastapi.security import HTTPBearer
from typing import List, Optional
from sqlalchemy.orm import Session

from app.core.database import get_db
from app.core.auth import get_current_user
from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate, UserResponse
from app.services.user_service import UserService
from app.schemas.pagination import PaginationParams, PaginatedResponse

router = APIRouter()
security = HTTPBearer()


@router.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    db: Session = Depends(get_db),
    user_service: UserService = Depends()
):
    """Create a new user"""
    try:
        user = await user_service.create_user(db, user_data)
        return UserResponse.from_orm(user)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))


@router.get("/users", response_model=PaginatedResponse[UserResponse])
async def get_users(
    pagination: PaginationParams = Depends(),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
    user_service: UserService = Depends()
):
    """Get all users with pagination"""
    users, total = await user_service.get_users(db, pagination)

    return PaginatedResponse(
        items=[UserResponse.from_orm(user) for user in users],
        total=total,
        page=pagination.page,
        size=pagination.size,
        pages=(total + pagination.size - 1) // pagination.size
    )


@router.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: str,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
    user_service: UserService = Depends()
):
    """Get user by ID"""
    user = await user_service.get_user_by_id(db, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")

    return UserResponse.from_orm(user)


@router.put("/users/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: str,
    user_data: UserUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
    user_service: UserService = Depends()
):
    """Update user"""
    user = await user_service.update_user(db, user_id, user_data)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")

    return UserResponse.from_orm(user)


@router.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: str,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
    user_service: UserService = Depends()
):
    """Delete user"""
    success = await user_service.delete_user(db, user_id)
    if not success:
        raise HTTPException(status_code=404, detail="User not found")
```

### 8.3 Template Service Go + Gin

#### Structure Complète

```go
// cmd/server/main.go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "github.com/go-redis/redis/v8"

    "app/internal/config"
    "app/internal/handlers"
    "app/internal/middleware"
    "app/internal/models"
    "app/internal/services"
    "app/internal/health"
    "app/internal/logging"
    "app/internal/metrics"
)

func main() {
    // Charger la configuration
    cfg := config.Load()

    // Initialiser le logger
    logger := logging.NewStructuredLogger()

    logger.Info("Starting Visiobook microservice", logrus.Fields{
        "service": cfg.ServiceName,
        "version": cfg.ServiceVersion,
        "port":    cfg.Port,
    })

    // Connexion base de données
    db, err := initDatabase(cfg.DatabaseURL)
    if err != nil {
        logger.Error("Failed to connect to database", err, logrus.Fields{})
        log.Fatal(err)
    }

    // Connexion Redis
    redisClient := initRedis(cfg.RedisURL)

    // Initialiser les services
    userService := services.NewUserService(db)
    healthService := health.NewHealthCheckService(db, redisClient)

    // Initialiser les handlers
    userHandler := handlers.NewUserHandler(userService)

    // Configurer Gin
    if cfg.Environment == "production" {
        gin.SetMode(gin.ReleaseMode)
    }

    router := gin.New()

    // Middleware globaux
    router.Use(middleware.LoggingMiddleware())
    router.Use(middleware.CorrelationIDMiddleware())
    router.Use(metrics.MetricsMiddleware())
    router.Use(middleware.CORSMiddleware())
    router.Use(middleware.SecurityHeadersMiddleware())

    // Routes de santé
    health.SetupHealthRoutes(router, healthService)

    // Routes API
    v1 := router.Group("/api/v1")
    {
        users := v1.Group("/users")
        {
            users.POST("", userHandler.CreateUser)
            users.GET("", middleware.AuthMiddleware(), userHandler.GetUsers)
            users.GET("/:id", middleware.AuthMiddleware(), userHandler.GetUserByID)
            users.PUT("/:id", middleware.AuthMiddleware(), userHandler.UpdateUser)
            users.DELETE("/:id", middleware.AuthMiddleware(), userHandler.DeleteUser)
        }
    }

    // Démarrer le serveur
    srv := &http.Server{
        Addr:    fmt.Sprintf(":%s", cfg.Port),
        Handler: router,
    }

    // Démarrage graceful
    go func() {
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            logger.Error("Failed to start server", err, logrus.Fields{})
            log.Fatal(err)
        }
    }()

    logger.Info("Server started successfully", logrus.Fields{
        "port": cfg.Port,
    })

    // Attendre le signal d'arrêt
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    logger.Info("Shutting down server...", logrus.Fields{})

    // Arrêt graceful avec timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        logger.Error("Server forced to shutdown", err, logrus.Fields{})
        log.Fatal(err)
    }

    logger.Info("Server exited", logrus.Fields{})
}

func initDatabase(databaseURL string) (*gorm.DB, error) {
    db, err := gorm.Open(postgres.Open(databaseURL), &gorm.Config{})
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }

    // Auto-migration
    err = db.AutoMigrate(&models.User{}, &models.Profile{})
    if err != nil {
        return nil, fmt.Errorf("failed to run migrations: %w", err)
    }

    return db, nil
}

func initRedis(redisURL string) *redis.Client {
    opt, err := redis.ParseURL(redisURL)
    if err != nil {
        log.Fatal("Failed to parse Redis URL:", err)
    }

    client := redis.NewClient(opt)

    // Test de connexion
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    _, err = client.Ping(ctx).Result()
    if err != nil {
        log.Fatal("Failed to connect to Redis:", err)
    }

    return client
}
```

#### Configuration

```go
// internal/config/config.go
package config

import (
    "os"
    "strconv"
)

type Config struct {
    // Service
    ServiceName    string
    ServiceVersion string
    Environment    string
    Port          string

    // Database
    DatabaseURL string
    RedisURL    string

    // Security
    JWTSecret     string
    JWTExpiresIn  string
    BCryptRounds  int

    // Monitoring
    PrometheusPort string
    LogLevel       string

    // External Services
    APIGatewayURL     string
    StorageServiceURL string
}

func Load() *Config {
    return &Config{
        // Service
        ServiceName:    getEnv("SERVICE_NAME", "unknown-service"),
        ServiceVersion: getEnv("SERVICE_VERSION", "1.0.0"),
        Environment:    getEnv("ENVIRONMENT", "development"),
        Port:          getEnv("PORT", "8080"),

        // Database
        DatabaseURL: getEnv("DATABASE_URL", ""),
        RedisURL:    getEnv("REDIS_URL", "redis://localhost:6379"),

        // Security
        JWTSecret:    getEnv("JWT_SECRET", ""),
        JWTExpiresIn: getEnv("JWT_EXPIRES_IN", "24h"),
        BCryptRounds: getEnvAsInt("BCRYPT_ROUNDS", 12),

        // Monitoring
        PrometheusPort: getEnv("PROMETHEUS_PORT", "9090"),
        LogLevel:       getEnv("LOG_LEVEL", "info"),

        // External Services
        APIGatewayURL:     getEnv("API_GATEWAY_URL", ""),
        StorageServiceURL: getEnv("STORAGE_SERVICE_URL", ""),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getEnvAsInt(key string, defaultValue int) int {
    if value := os.Getenv(key); value != "" {
        if intValue, err := strconv.Atoi(value); err == nil {
            return intValue
        }
    }
    return defaultValue
}
```

#### Handlers

```go
// internal/handlers/user_handler.go
package handlers

import (
    "net/http"
    "strconv"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"

    "app/internal/dto"
    "app/internal/services"
    "app/internal/logging"
)

type UserHandler struct {
    userService *services.UserService
}

func NewUserHandler(userService *services.UserService) *UserHandler {
    return &UserHandler{
        userService: userService,
    }
}

// CreateUser godoc
// @Summary Create a new user
// @Description Create a new user with the provided data
// @Tags users
// @Accept json
// @Produce json
// @Param user body dto.CreateUserRequest true "User data"
// @Success 201 {object} dto.UserResponse
// @Failure 400 {object} dto.ErrorResponse
// @Router /api/v1/users [post]
func (h *UserHandler) CreateUser(c *gin.Context) {
    var req dto.CreateUserRequest

    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       "Invalid request data",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    // Valider les données
    if err := req.Validate(); err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       err.Error(),
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    user, err := h.userService.CreateUser(req)
    if err != nil {
        logging.Logger.Error("Failed to create user", err, logrus.Fields{
            "correlation_id": c.GetString("correlation_id"),
            "email":         req.Email,
        })

        statusCode := http.StatusInternalServerError
        if err.Error() == "user already exists" {
            statusCode = http.StatusConflict
        }

        c.JSON(statusCode, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          statusCode,
                Message:       err.Error(),
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    response := dto.UserResponse{
        ID:        user.ID.String(),
        Email:     user.Email,
        Username:  user.Username,
        Role:      string(user.Role),
        CreatedAt: user.CreatedAt.Format(time.RFC3339),
        UpdatedAt: user.UpdatedAt.Format(time.RFC3339),
    }

    if user.Profile != nil {
        response.Profile = &dto.ProfileResponse{
            ID:        user.Profile.ID.String(),
            FirstName: user.Profile.FirstName,
            LastName:  user.Profile.LastName,
            Avatar:    user.Profile.Avatar,
            Bio:       user.Profile.Bio,
        }
    }

    c.JSON(http.StatusCreated, response)
}

// GetUsers godoc
// @Summary Get all users
// @Description Get all users with pagination
// @Tags users
// @Accept json
// @Produce json
// @Param page query int false "Page number" default(1)
// @Param limit query int false "Items per page" default(10)
// @Success 200 {object} dto.PaginatedResponse[dto.UserResponse]
// @Failure 500 {object} dto.ErrorResponse
// @Security BearerAuth
// @Router /api/v1/users [get]
func (h *UserHandler) GetUsers(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "10"))

    if page < 1 {
        page = 1
    }
    if limit < 1 || limit > 100 {
        limit = 10
    }

    pagination := dto.PaginationParams{
        Page:  page,
        Limit: limit,
    }

    users, total, err := h.userService.GetUsers(pagination)
    if err != nil {
        logging.Logger.Error("Failed to get users", err, logrus.Fields{
            "correlation_id": c.GetString("correlation_id"),
        })

        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusInternalServerError,
                Message:       "Failed to retrieve users",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    // Convertir en réponses
    userResponses := make([]dto.UserResponse, len(users))
    for i, user := range users {
        userResponses[i] = dto.UserResponse{
            ID:        user.ID.String(),
            Email:     user.Email,
            Username:  user.Username,
            Role:      string(user.Role),
            CreatedAt: user.CreatedAt.Format(time.RFC3339),
            UpdatedAt: user.UpdatedAt.Format(time.RFC3339),
        }

        if user.Profile != nil {
            userResponses[i].Profile = &dto.ProfileResponse{
                ID:        user.Profile.ID.String(),
                FirstName: user.Profile.FirstName,
                LastName:  user.Profile.LastName,
                Avatar:    user.Profile.Avatar,
                Bio:       user.Profile.Bio,
            }
        }
    }

    response := dto.PaginatedResponse[dto.UserResponse]{
        Items: userResponses,
        Total: total,
        Page:  page,
        Size:  limit,
        Pages: (total + int64(limit) - 1) / int64(limit),
    }

    c.JSON(http.StatusOK, response)
}

// GetUserByID godoc
// @Summary Get user by ID
// @Description Get a specific user by their ID
// @Tags users
// @Accept json
// @Produce json
// @Param id path string true "User ID"
// @Success 200 {object} dto.UserResponse
// @Failure 404 {object} dto.ErrorResponse
// @Security BearerAuth
// @Router /api/v1/users/{id} [get]
func (h *UserHandler) GetUserByID(c *gin.Context) {
    userIDStr := c.Param("id")
    userID, err := uuid.Parse(userIDStr)
    if err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       "Invalid user ID format",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    user, err := h.userService.GetUserByID(userID)
    if err != nil {
        logging.Logger.Error("Failed to get user", err, logrus.Fields{
            "correlation_id": c.GetString("correlation_id"),
            "user_id":       userIDStr,
        })

        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusInternalServerError,
                Message:       "Failed to retrieve user",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    if user == nil {
        c.JSON(http.StatusNotFound, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusNotFound,
                Message:       "User not found",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    response := dto.UserResponse{
        ID:        user.ID.String(),
        Email:     user.Email,
        Username:  user.Username,
        Role:      string(user.Role),
        CreatedAt: user.CreatedAt.Format(time.RFC3339),
        UpdatedAt: user.UpdatedAt.Format(time.RFC3339),
    }

    if user.Profile != nil {
        response.Profile = &dto.ProfileResponse{
            ID:        user.Profile.ID.String(),
            FirstName: user.Profile.FirstName,
            LastName:  user.Profile.LastName,
            Avatar:    user.Profile.Avatar,
            Bio:       user.Profile.Bio,
        }
    }

    c.JSON(http.StatusOK, response)
}

// UpdateUser godoc
// @Summary Update user
// @Description Update an existing user
// @Tags users
// @Accept json
// @Produce json
// @Param id path string true "User ID"
// @Param user body dto.UpdateUserRequest true "Updated user data"
// @Success 200 {object} dto.UserResponse
// @Failure 400 {object} dto.ErrorResponse
// @Failure 404 {object} dto.ErrorResponse
// @Security BearerAuth
// @Router /api/v1/users/{id} [put]
func (h *UserHandler) UpdateUser(c *gin.Context) {
    userIDStr := c.Param("id")
    userID, err := uuid.Parse(userIDStr)
    if err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       "Invalid user ID format",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    var req dto.UpdateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       "Invalid request data",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    user, err := h.userService.UpdateUser(userID, req)
    if err != nil {
        logging.Logger.Error("Failed to update user", err, logrus.Fields{
            "correlation_id": c.GetString("correlation_id"),
            "user_id":       userIDStr,
        })

        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusInternalServerError,
                Message:       "Failed to update user",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    if user == nil {
        c.JSON(http.StatusNotFound, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusNotFound,
                Message:       "User not found",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    response := dto.UserResponse{
        ID:        user.ID.String(),
        Email:     user.Email,
        Username:  user.Username,
        Role:      string(user.Role),
        CreatedAt: user.CreatedAt.Format(time.RFC3339),
        UpdatedAt: user.UpdatedAt.Format(time.RFC3339),
    }

    c.JSON(http.StatusOK, response)
}

// DeleteUser godoc
// @Summary Delete user
// @Description Delete an existing user
// @Tags users
// @Accept json
// @Produce json
// @Param id path string true "User ID"
// @Success 204
// @Failure 404 {object} dto.ErrorResponse
// @Security BearerAuth
// @Router /api/v1/users/{id} [delete]
func (h *UserHandler) DeleteUser(c *gin.Context) {
    userIDStr := c.Param("id")
    userID, err := uuid.Parse(userIDStr)
    if err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusBadRequest,
                Message:       "Invalid user ID format",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    err = h.userService.DeleteUser(userID)
    if err != nil {
        if err == gorm.ErrRecordNotFound {
            c.JSON(http.StatusNotFound, dto.ErrorResponse{
                Error: dto.ErrorDetail{
                    Code:          http.StatusNotFound,
                    Message:       "User not found",
                    CorrelationID: c.GetString("correlation_id"),
                    Timestamp:     time.Now().Unix(),
                },
            })
            return
        }

        logging.Logger.Error("Failed to delete user", err, logrus.Fields{
            "correlation_id": c.GetString("correlation_id"),
            "user_id":       userIDStr,
        })

        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error: dto.ErrorDetail{
                Code:          http.StatusInternalServerError,
                Message:       "Failed to delete user",
                CorrelationID: c.GetString("correlation_id"),
                Timestamp:     time.Now().Unix(),
            },
        })
        return
    }

    c.Status(http.StatusNoContent)
}
```

#### DTOs et Validation

```go
// internal/dto/user_dto.go
package dto

import (
    "errors"
    "regexp"
    "strings"
)

type CreateUserRequest struct {
    Email     string  `json:"email" binding:"required"`
    Username  string  `json:"username" binding:"required"`
    Password  string  `json:"password" binding:"required"`
    FirstName *string `json:"first_name,omitempty"`
    LastName  *string `json:"last_name,omitempty"`
}

type UpdateUserRequest struct {
    Email     *string `json:"email,omitempty"`
    Username  *string `json:"username,omitempty"`
    FirstName *string `json:"first_name,omitempty"`
    LastName  *string `json:"last_name,omitempty"`
}

type UserResponse struct {
    ID        string           `json:"id"`
    Email     string           `json:"email"`
    Username  string           `json:"username"`
    Role      string           `json:"role"`
    Profile   *ProfileResponse `json:"profile,omitempty"`
    CreatedAt string           `json:"created_at"`
    UpdatedAt string           `json:"updated_at"`
}

type ProfileResponse struct {
    ID        string  `json:"id"`
    FirstName *string `json:"first_name,omitempty"`
    LastName  *string `json:"last_name,omitempty"`
    Avatar    *string `json:"avatar,omitempty"`
    Bio       *string `json:"bio,omitempty"`
}

type PaginationParams struct {
    Page  int `json:"page"`
    Limit int `json:"limit"`
}

type PaginatedResponse[T any] struct {
    Items []T   `json:"items"`
    Total int64 `json:"total"`
    Page  int   `json:"page"`
    Size  int   `json:"size"`
    Pages int64 `json:"pages"`
}

type ErrorResponse struct {
    Error ErrorDetail `json:"error"`
}

type ErrorDetail struct {
    Code          int    `json:"code"`
    Message       string `json:"message"`
    CorrelationID string `json:"correlation_id"`
    Timestamp     int64  `json:"timestamp"`
}

// Validation methods
func (r *CreateUserRequest) Validate() error {
    // Email validation
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    if !emailRegex.MatchString(r.Email) {
        return errors.New("invalid email format")
    }

    // Username validation
    if len(r.Username) < 3 || len(r.Username) > 20 {
        return errors.New("username must be between 3 and 20 characters")
    }

    usernameRegex := regexp.MustCompile(`^[a-zA-Z0-9_-]+$`)
    if !usernameRegex.MatchString(r.Username) {
        return errors.New("username can only contain letters, numbers, underscores, and hyphens")
    }

    // Password validation
    if len(r.Password) < 8 {
        return errors.New("password must be at least 8 characters long")
    }

    // Name validation (if provided)
    if r.FirstName != nil && strings.TrimSpace(*r.FirstName) == "" {
        return errors.New("first name cannot be empty if provided")
    }

    if r.LastName != nil && strings.TrimSpace(*r.LastName) == "" {
        return errors.New("last name cannot be empty if provided")
    }

    return nil
}

func (r *UpdateUserRequest) Validate() error {
    // Email validation (if provided)
    if r.Email != nil {
        emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
        if !emailRegex.MatchString(*r.Email) {
            return errors.New("invalid email format")
        }
    }

    // Username validation (if provided)
    if r.Username != nil {
        if len(*r.Username) < 3 || len(*r.Username) > 20 {
            return errors.New("username must be between 3 and 20 characters")
        }

        usernameRegex := regexp.MustCompile(`^[a-zA-Z0-9_-]+$`)
        if !usernameRegex.MatchString(*r.Username) {
            return errors.New("username can only contain letters, numbers, underscores, and hyphens")
        }
    }

    // Name validation (if provided)
    if r.FirstName != nil && strings.TrimSpace(*r.FirstName) == "" {
        return errors.New("first name cannot be empty if provided")
    }

    if r.LastName != nil && strings.TrimSpace(*r.LastName) == "" {
        return errors.New("last name cannot be empty if provided")
    }

    return nil
}
```

#### Middleware

```go
// internal/middleware/auth.go
package middleware

import (
    "net/http"
    "strings"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"

    "app/internal/dto"
    "app/internal/config"
)

func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(http.StatusUnauthorized, dto.ErrorResponse{
                Error: dto.ErrorDetail{
                    Code:          http.StatusUnauthorized,
                    Message:       "Authorization header required",
                    CorrelationID: c.GetString("correlation_id"),
                    Timestamp:     time.Now().Unix(),
                },
            })
            c.Abort()
            return
        }

        // Extraire le token
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        if tokenString == authHeader {
            c.JSON(http.StatusUnauthorized, dto.ErrorResponse{
                Error: dto.ErrorDetail{
                    Code:          http.StatusUnauthorized,
                    Message:       "Invalid authorization format",
                    CorrelationID: c.GetString("correlation_id"),
                    Timestamp:     time.Now().Unix(),
                },
            })
            c.Abort()
            return
        }

        // Valider le token
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
            }
            return []byte(config.Load().JWTSecret), nil
        })

        if err != nil || !token.Valid {
            c.JSON(http.StatusUnauthorized, dto.ErrorResponse{
                Error: dto.ErrorDetail{
                    Code:          http.StatusUnauthorized,
                    Message:       "Invalid token",
                    CorrelationID: c.GetString("correlation_id"),
                    Timestamp:     time.Now().Unix(),
                },
            })
            c.Abort()
            return
        }

        // Extraire les claims
        if claims, ok := token.Claims.(jwt.MapClaims); ok {
            c.Set("user_id", claims["user_id"])
            c.Set("user_role", claims["role"])
        }

        c.Next()
    }
}

// internal/middleware/cors.go
package middleware

import (
    "github.com/gin-gonic/gin"
)

func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Credentials", "true")
        c.Header("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With, X-Correlation-ID")
        c.Header("Access-Control-Allow-Methods", "POST, OPTIONS, GET, PUT, DELETE")

        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }

        c.Next()
    }
}

// internal/middleware/security.go
package middleware

import (
    "github.com/gin-gonic/gin"
)

func SecurityHeadersMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("X-Content-Type-Options", "nosniff")
        c.Header("X-Frame-Options", "DENY")
        c.Header("X-XSS-Protection", "1; mode=block")
        c.Header("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        c.Header("Content-Security-Policy", "default-src 'self'")
        c.Header("Referrer-Policy", "strict-origin-when-cross-origin")

        c.Next()
    }
}
```

#### Tests

```go
// tests/unit/services/user_service_test.go
package services_test

import (
    "testing"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "gorm.io/

## 9. Checklist de Validation Avant Déploiement

### 9.1 Checklist Technique

#### Code Quality

```yaml
✅ Code Quality:
  - [ ] Linting passé sans erreurs (ESLint/Pylint/Golint)
  - [ ] Formatage uniforme (Prettier/Black/gofmt)
  - [ ] Types TypeScript stricts (si applicable)
  - [ ] Pas de console.log/print en production
  - [ ] Gestion d'erreurs complète
  - [ ] Documentation code à jour

✅ Tests:
  - [ ] Couverture tests > 80%
  - [ ] Tests unitaires passants
  - [ ] Tests d'intégration passants
  - [ ] Tests de contrat validés
  - [ ] Mocks correctement configurés
  - [ ] Tests E2E pour workflows critiques

✅ Sécurité:
  - [ ] Audit sécurité passé (npm audit/Snyk)
  - [ ] Variables sensibles dans secrets
  - [ ] HTTPS obligatoire
  - [ ] Validation des entrées
  - [ ] Authentification/autorisation
  - [ ] Headers de sécurité configurés
```

#### Infrastructure

```yaml
✅ Docker:
  - [ ] Dockerfile multi-stage optimisé
  - [ ] Image de base sécurisée
  - [ ] Utilisateur non-root
  - [ ] Health check configuré
  - [ ] Scan vulnérabilités passé
  - [ ] Taille image < 500MB

✅ Kubernetes:
  - [ ] Helm chart validé
  - [ ] Resource limits définis
  - [ ] Health checks configurés
  - [ ] Security context strict
  - [ ] Service account dédié
  - [ ] Network policies (si applicable)

✅ Monitoring:
  - [ ] Health endpoints (/health, /ready)
  - [ ] Métriques Prometheus exposées
  - [ ] Logs structurés JSON
  - [ ] Correlation ID propagé
  - [ ] Alertes configurées
  - [ ] Dashboard Grafana créé
```

### 9.2 Checklist Fonctionnel

#### API

```yaml
✅ API Design:
  - [ ] Convention de nommage respectée
  - [ ] Versioning implémenté (/api/v1/)
  - [ ] Documentation OpenAPI complète
  - [ ] Codes de statut HTTP corrects
  - [ ] Pagination implémentée
  - [ ] Rate limiting configuré

✅ Base de Données:
  - [ ] Migrations testées
  - [ ] Index optimisés
  - [ ] Contraintes de données
  - [ ] Backup strategy définie
  - [ ] Connection pooling configuré
  - [ ] Monitoring performances

✅ Performance:
  - [ ] Temps de réponse < objectifs
  - [ ] Throughput validé
  - [ ] Memory leaks vérifiés
  - [ ] Cache strategy implémentée
  - [ ] CDN configuré (si applicable)
  - [ ] Optimisations GPU (services IA)
```

### 9.3 Checklist Déploiement

#### Pre-Deployment

```yaml
✅ Préparation:
  - [ ] Environment variables configurées
  - [ ] Secrets Kubernetes créés
  - [ ] ConfigMaps à jour
  - [ ] DNS configuré
  - [ ] Certificats SSL valides
  - [ ] Backup base de données

✅ Validation:
  - [ ] Tests staging passants
  - [ ] Performance tests validés
  - [ ] Security scan passé
  - [ ] Rollback plan préparé
  - [ ] Monitoring configuré
  - [ ] Alertes activées
```

#### Post-Deployment

```yaml
✅ Vérification:
  - [ ] Service démarré correctement
  - [ ] Health checks OK
  - [ ] Métriques remontées
  - [ ] Logs visibles dans centralisés
  - [ ] Endpoints API accessibles
  - [ ] Intégration avec autres services
  - [ ] Performance conforme aux SLA
  - [ ] Alertes fonctionnelles
  - [ ] Documentation mise à jour
  - [ ] Équipe notifiée du déploiement
```

## 10. Conclusion et Prochaines Étapes

### 10.1 Résumé des Guidelines

Ce document de guidelines Visiobook établit les **standards obligatoires** pour le développement de tous les microservices de l'écosystème. Les points clés incluent :

#### Standards Techniques
- **Structure de projet standardisée** pour chaque stack technologique
- **Dockerfile multi-stage optimisés** pour la sécurité et performance
- **Variables d'environnement normalisées** pour la cohérence
- **ORM Prisma obligatoire** pour les services Node.js avec typage strict

#### Qualité et Tests
- **Couverture de tests minimale de 80%** avec tests unitaires et d'intégration
- **Mocking obligatoire** des dépendances externes
- **Contrats de test** entre services pour garantir l'interopérabilité
- **Tests E2E automatisés** pour les workflows critiques

#### Observabilité
- **Health checks standardisés** (`/health`, `/ready`, `/metrics`)
- **Logging structuré JSON** avec correlation IDs
- **Métriques Prometheus** obligatoires
- **KPIs spécifiques** par type de service

#### Infrastructure
- **CI/CD pipeline standardisé** avec GitHub Actions
- **Templates Helm charts** fournis par l'équipe infrastructure
- **Déploiement Blue-Green** pour la production
- **Monitoring et alerting** automatiques

### 10.2 Bénéfices Attendus

#### Pour les Développeurs
```yaml
Productivité:
  - Templates prêts à l'emploi
  - Standards clairs et documentés
  - Outils de développement unifiés
  - Debugging facilité avec correlation IDs

Qualité:
  - Tests automatisés obligatoires
  - Linting et formatage uniformes
  - Security scans intégrés
  - Code reviews standardisées
```

#### Pour l'Équipe
```yaml
Collaboration:
  - Conventions de nommage uniformes
  - Documentation automatique
  - Contrats d'interface clairs
  - Onboarding facilité

Maintenance:
  - Monitoring centralisé
  - Logs structurés
  - Alerting proactif
  - Rollback automatique
```

#### Pour le Projet
```yaml
Scalabilité:
  - Architecture microservices complète
  - Auto-scaling configuré
  - Performance optimisée
  - Ressources maîtrisées

Fiabilité:
  - Health checks complets
  - Circuit breakers
  - Retry policies
  - Backup automatique
```

### 10.3 Mise en Application

#### Phase 1 - Adoption Immédiate (Semaine 1)
```yaml
Actions Prioritaires:
  - [ ] Formation équipe sur les guidelines
  - [ ] Setup des templates dans les repositories
  - [ ] Configuration des outils de développement
  - [ ] Mise en place des pipelines CI/CD de base

Responsables:
  - Tech Lead: Formation et validation
  - DevOps: Templates infrastructure
  - Développeurs: Adoption des standards
```

#### Phase 2 - Implémentation (Semaines 2-3)
```yaml
Actions:
  - [ ] Application des standards aux services prioritaires
  - [ ] Mise en place du monitoring
  - [ ] Configuration des tests automatisés
  - [ ] Documentation des APIs

Validation:
  - Code reviews strictes
  - Tests de conformité
  - Métriques de qualité
  - Feedback équipe
```

#### Phase 3 - Optimisation (Semaines 4+)
```yaml
Actions:
  - [ ] Analyse des métriques de performance
  - [ ] Optimisation des pipelines
  - [ ] Amélioration continue des templates
  - [ ] Formation avancée équipe

Métriques de Succès:
  - Temps de développement réduit
  - Qualité code améliorée
  - Incidents production réduits
  - Satisfaction équipe élevée
```

### 10.4 Évolution et Maintenance

#### Versioning des Guidelines
```yaml
Stratégie:
  - Version majeure: Changements breaking
  - Version mineure: Nouvelles fonctionnalités
  - Version patch: Corrections et améliorations

Processus:
  - RFC pour changements majeurs
  - Review équipe pour modifications
  - Tests sur projet pilote
  - Déploiement progressif
```

#### Feedback et Amélioration Continue
```yaml
Mécanismes:
  - Retrospectives sprint
  - Surveys équipe trimestrielles
  - Métriques techniques automatiques
  - Benchmarking externe

Actions:
  - Mise à jour guidelines
  - Formation complémentaire
  - Outils additionnels
  - Process improvements
```

### 10.5 Support et Ressources

#### Documentation
- **Guidelines complètes** : Ce document
- **Templates de code** : Repositories GitHub
- **Exemples pratiques** : Projets de référence
- **FAQ développeurs** : Wiki interne

#### Formation
- **Onboarding nouveaux développeurs** : 2 jours
- **Sessions techniques mensuelles** : Approfondissement
- **Workshops pratiques** : Hands-on experience
- **Certification interne** : Validation des compétences

#### Support Technique
- **Canal Slack dédié** : `#visiobook-dev-guidelines`
- **Office hours** : Tech Lead disponible
- **Code reviews** : Validation conformité
- **Pair programming** : Accompagnement

### 10.6 Métriques de Succès

#### Métriques Techniques
```yaml
Qualité Code:
  - Couverture tests: > 80%
  - Complexité cyclomatique: < 10
  - Duplication code: < 5%
  - Vulnérabilités sécurité: 0

Performance:
  - Temps build CI/CD: < 10min
  - Temps déploiement: < 5min
  - MTTR incidents: < 30min
  - Disponibilité services: > 99.9%
```

#### Métriques Équipe
```yaml
Productivité:
  - Vélocité sprint: +20%
  - Temps onboarding: -50%
  - Bugs production: -60%
  - Satisfaction développeur: > 4/5

Collaboration:
  - Temps code review: < 24h
  - Conflits merge: -80%
  - Documentation à jour: 100%
  - Standards respectés: > 95%
```

---

## Annexes

### Annexe A - Liens Utiles

- **Architecture Visiobook** : `archi/visiobook-priority-architecture.md`
- **Templates GitHub** : `https://github.com/visiobook/templates`
- **Documentation Prisma** : `https://prisma.io/docs`
- **Helm Charts** : `infra-helm-charts/`
- **Monitoring Grafana** : `https://monitoring.visiobook.com`

### Annexe B - Contacts

- **Tech Lead** : `tech-lead@visiobook.com`
- **DevOps Team** : `devops@visiobook.com`
- **Support Développement** : `#visiobook-dev-support`

### Annexe C - Changelog

| Version | Date | Changements |
|---------|------|-------------|
| 1.0.0 | 2024-01-19 | Version initiale des guidelines |
| 1.1.0 | TBD | Ajouts basés sur feedback équipe |

---

**Document maintenu par** : Équipe Architecture Visiobook
**Dernière mise à jour** : 19 Janvier 2024
**Prochaine révision** : 19 Avril 2024

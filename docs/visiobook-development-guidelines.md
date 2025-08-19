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

### 1.3 Variables d'Environnement Standardisées

#### Variables Communes (Tous Services)

```bash
# Service Configuration
SERVICE_NAME=user-service
SERVICE_VERSION=1.0.0
PORT=8081
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:password@database-service:5432/visiobook
REDIS_URL=redis://redis-service:6379

# Security
JWT_SECRET=your-jwt-secret
JWT_EXPIRES_IN=24h
BCRYPT_ROUNDS=12

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=info
CORRELATION_ID_HEADER=x-correlation-id

# External Services
API_GATEWAY_URL=http://api-gateway-service:8080
STORAGE_SERVICE_URL=http://storage-service:8089
```

#### Variables Spécifiques par Service

```bash
# AI Analysis Service
CUDA_VISIBLE_DEVICES=0,1
MODEL_CACHE_DIR=/app/models
BATCH_SIZE=32
GPU_MEMORY_FRACTION=0.8

# Storage Service
AZURE_STORAGE_ACCOUNT=visiobook
AZURE_STORAGE_KEY=your-key
CDN_URL=https://cdn.visiobook.com

# Frontend Services
VITE_API_URL=https://api.visiobook.com
VITE_CDN_URL=https://cdn.visiobook.com
VITE_SENTRY_DSN=your-sentry-dsn
```

## 2. Gestion des Données et ORM

### 2.1 Prisma - Configuration Obligatoire (Services Node.js)

#### Schema Prisma Standardisé

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Modèle de base avec champs obligatoires
model BaseModel {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  version   Int      @default(1)

  @@map("base_model")
}

// Exemple : User Model
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  username  String   @unique
  password  String
  role      UserRole @default(USER)
  profile   Profile?
  projects  Project[]

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  version   Int      @default(1)

  @@map("users")
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}

// Relation Profile
model Profile {
  id       String  @id @default(cuid())
  userId   String  @unique @map("user_id")
  user     User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  firstName String? @map("first_name")
  lastName  String? @map("last_name")
  avatar    String?
  bio       String?

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("profiles")
}
```

#### Service Pattern avec Prisma

```typescript
// src/services/user.service.ts
import { PrismaClient, User, Prisma } from '@prisma/client';
import { CreateUserDto, UpdateUserDto } from '../types/user.types';

export class UserService {
  constructor(private prisma: PrismaClient) {}

  async createUser(data: CreateUserDto): Promise<User> {
    try {
      return await this.prisma.user.create({
        data: {
          ...data,
          profile: {
            create: {
              firstName: data.firstName,
              lastName: data.lastName,
            },
          },
        },
        include: {
          profile: true,
        },
      });
    } catch (error) {
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        if (error.code === 'P2002') {
          throw new Error('User already exists');
        }
      }
      throw error;
    }
  }

  async getUserById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        profile: true,
        projects: true,
      },
    });
  }

  async updateUser(id: string, data: UpdateUserDto): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data: {
        ...data,
        version: {
          increment: 1,
        },
      },
      include: {
        profile: true,
      },
    });
  }

  async deleteUser(id: string): Promise<void> {
    await this.prisma.user.delete({
      where: { id },
    });
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

### 3.2 Implémentation avec Express.js

```typescript
// src/routes/user.routes.ts
import { Router } from 'express';
import { UserController } from '../controllers/user.controller';
import { validateRequest } from '../middleware/validation.middleware';
import { authenticate } from '../middleware/auth.middleware';
import { CreateUserSchema, UpdateUserSchema } from '../types/user.types';

const router = Router();
const userController = new UserController();

// Routes publiques
router.post(
  '/api/v1/users',
  validateRequest(CreateUserSchema),
  userController.createUser
);

// Routes protégées
router.use('/api/v1/users', authenticate);

router.get('/api/v1/users', userController.getUsers);
router.get('/api/v1/users/:id', userController.getUserById);
router.put(
  '/api/v1/users/:id',
  validateRequest(UpdateUserSchema),
  userController.updateUser
);
router.delete('/api/v1/users/:id', userController.deleteUser);

// Sous-ressources
router.get('/api/v1/users/:id/projects', userController.getUserProjects);
router.post('/api/v1/users/:id/projects', userController.createUserProject);

export default router;
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

### 5.2 Logging Structuré

#### Configuration Winston

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

## 7. Choix Technologiques Approuvés

### 7.1 Stack Technologique par Type de Service

#### Services Backend (Node.js)

```yaml
Stack Approuvée:
  Runtime: Node.js 18.18+ LTS
  Language: TypeScript 5.2+
  Framework: NestJS 10+ ou Express.js 4.18+
  ORM: Prisma 5.6+ (obligatoire)
  Validation: Zod 3.22+ ou class-validator
  Testing: Jest 29+ + Supertest
  Documentation: Swagger/OpenAPI 3.0+

Librairies Recommandées:
  HTTP Client: Axios 1.6+
  Logging: Winston 3.11+
  Monitoring: Prometheus client
  Security: Helmet, bcrypt, jsonwebtoken
  Utilities: Lodash, date-fns, uuid

Librairies Interdites:
  - Moment.js (utiliser date-fns)
  - Request (deprecated, utiliser Axios)
  - Mongoose (utiliser Prisma)
```

#### Services IA (Python)

```yaml
Stack Approuvée:
  Runtime: Python 3.11+
  Framework: FastAPI 0.104+
  ML Framework: PyTorch 2.1+ + Lightning
  NLP: Transformers 4.35+, spaCy 3.7+
  Data: NumPy 1.24+, Pandas 2.1+
  Testing: pytest 7.4+ + pytest-asyncio
  Documentation: FastAPI auto-docs

Librairies Recommandées:
  HTTP Client: httpx, aiohttp
  Validation: Pydantic 2.4+
  Monitoring: prometheus-client
  GPU: CUDA 12.1+, cuDNN 8.9+
  Vector DB: pinecone-client, pgvector

Optimisations GPU:
  - Model quantization (FP16/INT8)
  - Dynamic batching
  - Memory mapping
  - Pipeline parallèle
```

#### Services Infrastructure (Go)

```yaml
Stack Approuvée:
  Runtime: Go 1.21+
  Framework: Gin 1.9+ ou Fiber 2.50+
  Database: GORM 1.25+ ou sqlx
  Testing: Testify + GoMock
  Documentation: Swagger avec go-swagger

Librairies Recommandées:
  HTTP Client: resty
  Logging: logrus, zap
  Monitoring: prometheus/client_golang
  Configuration: viper
  Utilities: uuid, validator

Performance:
  - Connection pooling
  - Context propagation
  - Graceful shutdown
  - Memory optimization
```

#### Frontend (Vue.js)

```yaml
Stack Approuvée:
  Framework: Vue.js 3.3+ + TypeScript 5.0+
  Build Tool: Vite 4.0+
  State: Pinia + Vue Query (TanStack)
  UI: Vuetify 3+ + Material Design 3
  Router: Vue Router 4+
  Testing: Vitest + Vue Testing Library

Librairies Recommandées:
  HTTP Client: Axios + Vue Query
  Forms: VeeValidate + Yup
  Utils: date-fns, lodash-es
  Icons: Material Design Icons
  Charts: Chart.js + vue-chartjs

Performance:
  - Code splitting automatique
  - Lazy loading composants
  - Service Worker
  - Image optimization
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

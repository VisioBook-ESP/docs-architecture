# Visiobook Development Guidelines - Résumé Exécutif

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

Tous les microservices doivent suivre cette structure générique :

```
microservice-name/
├── src/                     # Code source
│   ├── controllers/         # Contrôleurs API
│   ├── services/           # Logique métier
│   ├── models/             # Modèles de données
│   ├── middleware/         # Middlewares
│   ├── utils/              # Utilitaires
│   ├── config/             # Configuration
│   └── types/              # Types/Interfaces
├── tests/
│   ├── unit/               # Tests unitaires
│   ├── integration/        # Tests d'intégration
│   └── mocks/              # Mocks pour tests
├── docker/
│   └── Dockerfile          # Image de production
├── k8s/                    # Configuration Kubernetes
├── .env.example            # Variables d'environnement
└── README.md
```

### 1.2 Configuration Helm Centralisée Sécurisée

#### Architecture de Configuration avec Contrôle d'Accès

L'architecture Visiobook utilise une approche **centralisée sécurisée** avec le repository `infra-helm-charts` pour garantir la gouvernance et le contrôle d'accès aux ressources critiques.

**Environnements et Accès :**

- **Développement Local (Minikube)** : Accès développeurs complet, ressources CPU/RAM limitées, pas de GPU
- **Staging** : Accès développeurs lecture seule via ArgoCD UI, GPU limités pour tests
- **Production** : Accès DevOps uniquement, GPU production complets, secrets chiffrés

#### Workflow de Déploiement Sécurisé

1. **Développement Local** : Développeur modifie code → Test sur Minikube → Push code → CI/CD build image
2. **Staging (Automatique)** : ArgoCD détecte nouvelle image → Déploie automatiquement → Tests d'intégration
3. **Production (Manuel)** : DevOps approuve → ArgoCD déploie → Monitoring et rollback si nécessaire

#### Gestion des Secrets Sécurisée

- **Développement** : Secrets plain text (non-sensibles), accès développeurs complet
- **Staging** : Sealed Secrets chiffrés, accès développeurs lecture seule
- **Production** : Azure Key Vault + External Secrets Operator, accès DevOps uniquement

### 1.3 Gestion des Données

#### Stratégie Multi-ORM

L'architecture adopte une approche **hybride** :

1. **Phase de développement** : Chaque microservice utilise sa base de données locale avec l'ORM de son choix
2. **Phase de consolidation** : Les schémas sont migrés vers le `database-service` centralisé
3. **Phase de production** : Les microservices se connectent au `database-service` via des contrats standardisés

#### Standards Obligatoires (Indépendants de l'ORM)

**Conventions de Nommage :**
- Tables : snake_case (users, user_profiles, project_files)
- Colonnes : snake_case (created_at, updated_at, user_id)
- Index : idx_table_column
- Contraintes : fk_table_column

**Champs Obligatoires :**
- id : UUID v4 (primary key)
- created_at : timestamp with timezone
- updated_at : timestamp with timezone
- version : integer (optimistic locking)

**Types de Données Standardisés :**
- ID : UUID (36 chars)
- Email : VARCHAR(255)
- Timestamps : TIMESTAMPTZ
- JSON : JSONB (PostgreSQL)
- Texte : TEXT (contenu long)

## 2. Convention de Nommage des Routes API

### 2.1 Format Standardisé

**Structure des URLs :**
```
/api/v{version}/{service-name}/{resource}[/{id}][/{sub-resource}]
```

**Exemples :**
- GET `/api/v1/users` - Liste des utilisateurs
- POST `/api/v1/users` - Créer un utilisateur
- GET `/api/v1/users/{id}` - Obtenir un utilisateur
- PUT `/api/v1/users/{id}` - Mettre à jour un utilisateur
- DELETE `/api/v1/users/{id}` - Supprimer un utilisateur
- GET `/api/v1/users/{id}/projects` - Projets d'un utilisateur

### 2.2 Conventions de Nommage

**Services :**
- users (user-service)
- projects (project-service)
- storage (storage-service)
- analysis (ai-analysis-service)
- media (media-generation-service)

**Resources :**
- Toujours au pluriel
- Kebab-case pour les mots composés
- Pas de verbes dans les noms de ressources

**Actions HTTP :**
- GET : Récupération (liste ou détail)
- POST : Création
- PUT : Mise à jour complète
- PATCH : Mise à jour partielle
- DELETE : Suppression

**Query Parameters :**
- page, limit : Pagination
- sort : Tri (ex: sort=createdAt:desc)
- filter : Filtrage (ex: filter[status]=active)
- include : Relations à inclure

## 3. Stratégie de Tests et Qualité

### 3.1 Exigences de Couverture

**Couverture minimale obligatoire : 80%**

**Types de tests obligatoires :**
- Tests unitaires : Logique métier isolée
- Tests d'intégration : Interaction entre composants
- Tests de contrat : Validation des APIs entre services
- Tests E2E : Workflows critiques complets

### 3.2 Stratégie de Mocking

**Mocking obligatoire pour :**
- Services externes (APIs tierces)
- Base de données (tests unitaires)
- Services internes (isolation des tests)
- Ressources système (fichiers, réseau)

**Principes :**
- Mocks configurables par environnement
- Données de test réalistes
- Validation des contrats d'interface
- Isolation complète des tests unitaires

### 3.3 Contrats de Test entre Services

**Définition obligatoire :**
- Endpoints API standardisés
- Formats de réponse validés
- Codes de statut HTTP cohérents
- Health checks uniformes

## 4. Monitoring et Observabilité

### 4.1 Health Checks Standardisés

**Endpoints obligatoires :**
- `/health` : État général du service
- `/ready` : Prêt à recevoir du trafic
- `/metrics` : Métriques Prometheus

**Vérifications requises :**
- Connectivité base de données
- Services externes critiques
- Configuration valide
- Migrations à jour

### 4.2 Logging Structuré

**Format obligatoire : JSON structuré**

**Champs requis :**
- timestamp : ISO 8601
- level : debug/info/warn/error
- message : Description claire
- service : Nom du service
- version : Version du service
- correlation_id : Traçabilité des requêtes

**Correlation IDs :**
- Propagation obligatoire entre services
- Headers : x-correlation-id ou x-request-id
- Logging de toutes les requêtes HTTP
- Traçabilité des erreurs

### 4.3 Métriques Prometheus

**Métriques obligatoires :**

**HTTP :**
- Durée des requêtes (histogramme)
- Nombre total de requêtes (compteur)
- Erreurs HTTP (compteur)

**Base de Données :**
- Connexions actives (gauge)
- Durée des requêtes (histogramme)
- Erreurs de requêtes (compteur)

**Business :**
- Opérations métier (compteur)
- Durée des opérations (histogramme)
- KPIs spécifiques par service

### 4.4 KPIs Obligatoires par Service

**Performance :**
- Temps de réponse moyen
- P95 et P99 des temps de réponse
- Throughput (requêtes/seconde)

**Fiabilité :**
- Uptime (pourcentage)
- Taux d'erreur (pourcentage)
- Taux de succès (pourcentage)

**Ressources :**
- Utilisation CPU (pourcentage)
- Utilisation mémoire (pourcentage)
- Utilisation disque (pourcentage)

**Business (spécifiques par service) :**
- user-service : Utilisateurs actifs, nouvelles inscriptions
- project-service : Projets créés, projets complétés
- ai-analysis-service : Requêtes d'analyse, utilisation GPU
- storage-service : Fichiers uploadés, utilisation stockage

## 5. CI/CD et Infrastructure

### 5.1 Pipeline CI/CD Standardisé

**Étapes obligatoires :**

**1. Test :**
- Linting et formatage
- Vérification des types
- Tests unitaires et d'intégration
- Couverture de code > 80%

**2. Sécurité :**
- Audit des dépendances
- Scan de vulnérabilités
- Analyse de sécurité du code

**3. Build :**
- Construction d'image Docker multi-stage
- Scan de vulnérabilités de l'image
- Push vers registry sécurisé

**4. Déploiement :**
- Staging automatique (branche develop)
- Production manuel avec approbation (branche main)
- Tests de fumée post-déploiement

### 5.2 Stratégie de Déploiement

**Staging :**
- Déploiement automatique
- Tests d'intégration complets
- Validation des performances

**Production :**
- Déploiement Blue-Green
- Approbation manuelle DevOps
- Rollback automatique en cas d'échec
- Monitoring continu post-déploiement

### 5.3 Gestion des Environnements

**Développement :**
- Minikube local
- Accès développeur complet
- Ressources limitées
- Secrets non-sensibles

**Staging :**
- Cluster partagé
- Accès lecture seule développeurs
- Tests de charge
- Secrets chiffrés

**Production :**
- Cluster dédié
- Accès DevOps uniquement
- Haute disponibilité
- Secrets Azure Key Vault

## 6. Standards Multi-Stack et Gouvernance

### 6.1 Stacks Technologiques Approuvées

**Node.js + TypeScript :**
- Services CRUD traditionnels
- APIs REST/GraphQL
- Framework : NestJS ou Express.js
- ORM : Prisma (recommandé)

**Python + FastAPI :**
- Services IA et Machine Learning
- Traitement de données
- APIs haute performance
- ORM : SQLAlchemy

**Go :**
- Services haute performance
- Gateways et proxies
- Microservices légers
- Framework : Gin ou Fiber

**Vue.js + TypeScript :**
- Applications web
- Dashboards admin
- Interfaces utilisateur
- Build : Vite

### 6.2 Standards Transversaux Obligatoires

**API Design :**
- OpenAPI 3.0+ pour tous
- Convention REST standardisée
- Codes de statut HTTP uniformes
- Headers de sécurité identiques

**Monitoring :**
- Health checks uniformes
- Métriques Prometheus format standard
- Logging JSON structuré
- Correlation IDs propagés

**Sécurité :**
- JWT tokens standardisés
- HTTPS obligatoire
- Headers sécurité uniformes
- Validation des entrées

**Infrastructure :**
- Dockerfiles multi-stage
- Helm charts adaptables
- Variables d'environnement cohérentes
- CI/CD pipelines similaires

### 6.3 Processus d'Approbation Nouvelles Stacks

**Critères d'Évaluation :**
- Maturité de l'écosystème
- Performance et scalabilité
- Sécurité et maintenance
- Expertise équipe disponible
- Compatibilité infrastructure

**Processus :**
1. Proposition motivée par l'équipe
2. Évaluation technique par l'architecture
3. Proof of Concept sur service non-critique
4. Validation des standards transversaux
5. Formation équipe et documentation
6. Approbation finale et adoption

## 7. Checklist de Validation Avant Déploiement

### 7.1 Code Quality

- [ ] Linting passé sans erreurs
- [ ] Formatage uniforme appliqué
- [ ] Types stricts validés (si applicable)
- [ ] Pas de logs de debug en production
- [ ] Gestion d'erreurs complète
- [ ] Documentation code à jour

### 7.2 Tests

- [ ] Couverture tests > 80%
- [ ] Tests unitaires passants
- [ ] Tests d'intégration passants
- [ ] Tests de contrat validés
- [ ] Mocks correctement configurés
- [ ] Tests E2E pour workflows critiques

### 7.3 Sécurité

- [ ] Audit sécurité passé
- [ ] Variables sensibles dans secrets
- [ ] HTTPS obligatoire
- [ ] Validation des entrées
- [ ] Authentification/autorisation
- [ ] Headers de sécurité configurés

### 7.4 Infrastructure

- [ ] Dockerfile optimisé
- [ ] Image de base sécurisée
- [ ] Utilisateur non-root
- [ ] Health check configuré
- [ ] Scan vulnérabilités passé
- [ ] Helm chart validé
- [ ] Resource limits définis
- [ ] Security context strict

### 7.5 Monitoring

- [ ] Health endpoints fonctionnels
- [ ] Métriques Prometheus exposées
- [ ] Logs structurés JSON
- [ ] Correlation ID propagé
- [ ] Alertes configurées
- [ ] Dashboard créé

### 7.6 Déploiement

- [ ] Environment variables configurées
- [ ] Secrets Kubernetes créés
- [ ] ConfigMaps à jour
- [ ] DNS configuré
- [ ] Certificats SSL valides
- [ ] Tests staging passants
- [ ] Performance validée
- [ ] Rollback plan préparé

## 8. Métriques de Succès

### 8.1 Métriques Techniques

**Qualité Code :**
- Couverture tests : > 80%
- Complexité cyclomatique : < 10
- Duplication code : < 5%
- Vulnérabilités sécurité : 0

**Performance :**
- Temps build CI/CD : < 10min
- Temps déploiement : < 5min
- MTTR incidents : < 30min
- Disponibilité services : > 99.9%

### 8.2 Métriques Équipe

**Productivité :**
- Vélocité sprint : +20%
- Temps onboarding : -50%
- Bugs production : -60%
- Satisfaction développeur : > 4/5

**Collaboration :**
- Temps code review : < 24h
- Conflits merge : -80%
- Documentation à jour : 100%
- Standards respectés : > 95%

## 9. Support et Ressources

### 9.1 Documentation

- **Guidelines complètes** : `visiobook-development-guidelines.md`
- **Templates de code** : Repositories GitHub
- **Exemples pratiques** : Projets de référence
- **FAQ développeurs** : Wiki interne

### 9.2 Formation

- **Onboarding nouveaux développeurs** : 2 jours
- **Sessions techniques mensuelles** : Approfondissement
- **Workshops pratiques** : Hands-on experience
- **Certification interne** : Validation des compétences

### 9.3 Support Technique

- **Canal Slack dédié** : `#visiobook-dev-guidelines`
- **Office hours** : Tech Lead disponible
- **Code reviews** : Validation conformité
- **Pair programming** : Accompagnement

---

## Conclusion

Ces guidelines établissent les **standards obligatoires** pour garantir la cohérence, la qualité et l'interopérabilité de l'écosystème Visiobook. Pour les détails d'implémentation, exemples de code et configurations spécifiques, consultez le document complet `visiobook-development-guidelines.md`.

**Document maintenu par** : Équipe Architecture Visiobook
**Version** : 1.0.0
**Dernière mise à jour** : 26 Août 2025

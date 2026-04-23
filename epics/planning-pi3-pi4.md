# Planning Prévisionnel — PI 3.3 & PI 4

> Dernière mise à jour : 2026-04-23
> Basé sur le [Diagramme de Gantt](./gantt.html) et la vélocité mesurée de l'équipe

---

## Situation actuelle (23/04/2026)

| Métrique | Valeur |
|----------|--------|
| **PI en cours** | PI 3 — Itération 3.3 (18/04 → 17/05) |
| **SP livrés** | 622 / 854 (73%) |
| **Issues fermées** | 164 / 211 (78%) |
| **Prochaine milestone** | Application complète — 17/05/2026 |

---

## Itération 3.3 — En cours (18/04 → 17/05/2026)

> **Objectif** : Finaliser les interfaces, lancer l'assemblage storyboard

### Nouvelles issues

| Repo | Issue | Titre | SP | Heures | Assigné |
|------|-------|-------|:--:|:------:|---------|
| ai-storyboard-assembly | #1 | Setup FastAPI + FFmpeg + NATS | 8 | 3.5h | VictorVattierEpitech |
| ai-storyboard-assembly | #2 | Assemblage vidéo (images + audio → MP4) | 13 | 6h | VictorVattierEpitech |
| ai-storyboard-assembly | #3 | NATS workflow (storyboard.completed) | 8 | 3.5h | VictorVattierEpitech |
| ai-storyboard-assembly | #4 | Tests + Helm chart + déploiement | 5 | 2.5h | VictorVattierEpitech |

### Issues existantes à terminer

| Repo | Issue | Titre | SP | Assigné |
|------|-------|-------|:--:|---------|
| mobile-flutter | #39 | Phase 13: Intégration API | 8 | Marinegyt |
| mobile-flutter | #40 | Phase 14: Profil Utilisateur | 8 | Marinegyt |
| core-user | #12 | Auth – Refresh Tokens | 5 | Marinegyt |
| ai-media-gen | #24 | Phase 4 - NATS workflow | 8 | VictorVattierEpitech |
| core-project | #31 | Phase 4 - Polish | 8 | Romain-Ber |

**Total itération 3.3** : ~71 SP — ~28h de dev

---

## PI 4 — Qualité, Optimisation et Déploiement (18/05 → 17/08/2026)

### Itération 4.1 (18/05 → 17/06/2026)

> **Objectif** : Système de paiement E2E, notifications, début des tests fonctionnels

| Repo | Issue | Titre | SP | Heures | Assigné |
|------|-------|-------|:--:|:------:|---------|
| core-notification | #1 | Setup NestJS + transports (email, push) | 8 | 3.5h | FlorianBernier |
| core-notification | #2 | NATS consumer → notifications | 8 | 3.5h | FlorianBernier |
| core-notification | #3 | Tests + Helm chart | 5 | 2.5h | FlorianBernier |
| core-payment | #28 | Intégration Stripe E2E (mobile + web) | 8 | 3.5h | FlorianBernier |
| web-user-portal | #16 | Stripe Checkout + affichage plans | 8 | 3.5h | camilogzlez |
| docs-architecture | #1 | Plan de test E2E cross-services | 13 | 6h | Marinegyt |
| mobile-flutter | #52 | Intégrer flutter_stripe + plans/quotas | 8 | 3.5h | Marinegyt |
| web-user-portal | #12 | Phase 3 - Auth | 8 | 3.5h | camilogzlez |
| web-user-portal | #13 | Phase 4 - Dashboard + Projets | 8 | 3.5h | camilogzlez |

**Total itération 4.1** : ~74 SP — ~30h de dev

### Itération 4.2 (18/06 → 17/07/2026)

> **Objectif** : Tests spécifiques IA, optimisation performances, coverage

| Repo | Issue | Titre | SP | Heures | Assigné |
|------|-------|-------|:--:|:------:|---------|
| ai-analysis | #43 | Tests qualité IA — validation output LLM | 8 | 3.5h | Romain-Ber |
| ai-analysis | #42 | Phase 6 - Tests + Monitoring | 8 | 3.5h | Romain-Ber |
| ai-media-gen | #28 | Tests qualité images (FLUX/LTX) | 8 | 3.5h | VictorVattierEpitech |
| ai-media-gen | #26 | Phase 6 - Tests (unit + GPU) | 8 | 3.5h | VictorVattierEpitech |
| infra-helm | #88 | Optimisation perfs (HPA, caching, load test) | 8 | 3.5h | vicous6 |
| core-payment | #26 | Phase 6 - Tests (unit + e2e) | 8 | 3.5h | FlorianBernier |
| mobile-flutter | #44 | Tests : atteindre 80% de couverture | 8 | 3.5h | Marinegyt |
| web-user-portal | #14 | Phase 5 - Profil + Abonnements | 8 | 3.5h | camilogzlez |

**Total itération 4.2** : ~64 SP — ~26h de dev

### Itération 4.3 (18/07 → 17/08/2026)

> **Objectif** : Mise en production, monitoring, beta test, polish final

| Repo | Issue | Titre | SP | Heures | Assigné |
|------|-------|-------|:--:|:------:|---------|
| infra-helm | #89 | Mise en production (DNS, TLS, visiobook.cloud) | 8 | 3.5h | vicous6 |
| infra-helm | #90 | Monitoring & Alerting (Grafana, alertes) | 8 | 3.5h | vicous6 |
| mobile-flutter | #69 | Tests UX — beta testing | 8 | 3.5h | Marinegyt |
| web-user-portal | #17 | Tests UX web | 5 | 2.5h | camilogzlez |
| web-user-portal | #15 | Phase 6 - Tests + i18n | 8 | 3.5h | camilogzlez |
| core-api-gateway | #9-#11 | Rate limiting, logging, tests | 18 | 7h | vicous6 |
| core-payment | #27 | Phase 7 - Production (Prometheus) | 13 | 6h | FlorianBernier |

**Total itération 4.3** : ~68 SP — ~27h de dev

---

## Jalons

| Date | Jalon | Statut |
|------|-------|--------|
| 17/11/2025 | Infrastructure opérationnelle | ✅ Atteint |
| 17/02/2026 | Génération d'images et animations | ✅ Atteint |
| 17/05/2026 | Application complète | ⏳ En cours |
| **17/08/2026** | **Application en production** | 🔮 Planifié |

---

## Charge par personne — PI 4

| Membre | SP planifiés PI 4 | Heures | Semaines (1j/sem) |
|--------|:-----------------:|:------:|:------------------:|
| **vicous6** | 50 | 20h | ~3 semaines |
| **Marinegyt** | 45 | 18h | ~3 semaines |
| **FlorianBernier** | 50 | 20h | ~3 semaines |
| **camilogzlez** | 45 | 18h | ~3 semaines |
| **VictorVattierEpitech** | 24 | 10h | ~1.5 semaines |
| **Romain-Ber** | 16 | 7h | ~1 semaine |
| **TasseritNicolas** | 0 | 0h | — |
| **Camserho** | 0 | 0h | — |

> **Note** : Camserho et TasseritNicolas ont terminé leurs tâches (content-ingestion 100%, infra CI terminée). Ils peuvent être mobilisés en renfort si besoin.

---

## Risques identifiés

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Assemblage storyboard complexe (FFmpeg + GPU) | Retard PI 3.3 | VictorVattierEpitech a l'expertise GPU, enablers infra déjà en place |
| Intégration Stripe E2E (3 services) | Retard PI 4.1 | Le backend payment est à 66%, réduire le scope au flux checkout simple |
| Tests IA subjectifs (qualité images/prompts) | Pas de métrique claire | Définir des critères objectifs avant de commencer (benchmark dataset) |
| Charge inégale (Camserho/TasseritNicolas à 0) | Sous-utilisation | Les repositionner sur les tests E2E ou le monitoring |

---

## Liens

- [Roadmap Globale](https://github.com/orgs/VisioBook-ESP/projects/5)
- [Rapport Story Points](./story-points-report.md)
- [Diagramme de Gantt](./gantt.html)
- [User Stories](./user_stories.md)

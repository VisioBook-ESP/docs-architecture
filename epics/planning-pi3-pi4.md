# Planning Prévisionnel — PI 3.3 & PI 4

> Dernière mise à jour : 2026-05-07
> Basé sur le [Diagramme de Gantt](./gantt.html) et la vélocité mesurée de l'équipe

---

## Situation actuelle (07/05/2026)

| Métrique | Valeur |
|----------|--------|
| **PI en cours** | PI 3 — Itération 3.3 (18/04 → 17/05) |
| **SP livrés** | 936 / 1167 (**80%**) |
| **Issues fermées** | 199 / 254 (**78%**) |
| **Prochaine milestone** | Application complète — 17/05/2026 |

---

## Itération 3.3 — En cours (18/04 → 17/05/2026)

> **Objectif** : Finaliser les interfaces, lancer l'assemblage storyboard

### Issues restantes

| Repo | Issue | Titre | SP | Assigné |
|------|-------|-------|:--:|---------|
| mobile-flutter | #39 | Phase 13: Intégration API | 8 | Marinegyt |
| mobile-flutter | #40 | Phase 14: Profil Utilisateur | 8 | Marinegyt |
| core-user | #12 | Auth – Refresh Tokens | 5 | Marinegyt |
| ai-media-gen | #24 | Phase 4 - NATS workflow | 8 | VictorVattierEpitech |
| core-project | #31 | Phase 4 - Polish | 8 | Romain-Ber |
| marketing | #7 | Landing Page visiobook.cloud | 8 | Marinegyt |
| marketing | #8 | Screenshots & Mockups | 5 | Marinegyt |

**Total restant itération 3.3** : ~50 SP

---

## PI 4 — Qualité, Optimisation et Déploiement (18/05 → 17/08/2026)

### Itération 4.1 (18/05 → 17/06/2026)

> **Objectif** : Paiement E2E, notifications, tests fonctionnels

| Repo | Issue | Titre | SP | Assigné |
|------|-------|-------|:--:|---------|
| core-payment | #26 | Phase 6 - Tests + Coverage | 8 | FlorianBernier |
| core-payment | #27 | Phase 7 - Production | 13 | FlorianBernier |
| web-user-portal | #12 | Phase 3 - Auth | 8 | camilogzlez |
| web-user-portal | #13 | Phase 4 - Dashboard + Projets | 8 | camilogzlez |
| ai-analysis | #42 | Phase 6 - Tests + Monitoring | 8 | Romain-Ber |
| marketing | #9 | Stratégie de lancement | 8 | Marinegyt |
| marketing | #10 | Contenu réseaux sociaux | 5 | Marinegyt |

**Total itération 4.1** : ~58 SP

### Itération 4.2 (18/06 → 17/07/2026)

> **Objectif** : Tests IA, optimisation, coverage

| Repo | Issue | Titre | SP | Assigné |
|------|-------|-------|:--:|---------|
| ai-media-gen | #25 | Phase 5 - K8s + Istio | 5 | VictorVattierEpitech |
| ai-media-gen | #26 | Phase 6 - Tests GPU | 8 | VictorVattierEpitech |
| web-user-portal | #14 | Phase 5 - Profil + Abonnements | 8 | camilogzlez |
| core-api-gateway | #9 | Rate limiting + CORS | 5 | vicous6 |
| core-api-gateway | #10 | Logging + Monitoring | 5 | vicous6 |
| marketing | #11 | ASO — Fiches stores | 5 | Marinegyt |
| marketing | #12 | Documentation utilisateur | 8 | Marinegyt |

**Total itération 4.2** : ~44 SP

### Itération 4.3 (18/07 → 17/08/2026)

> **Objectif** : Mise en production, monitoring, polish final

| Repo | Issue | Titre | SP | Assigné |
|------|-------|-------|:--:|---------|
| core-api-gateway | #11 | Tests + Documentation | 8 | vicous6 |
| web-user-portal | #15 | Phase 6 - Tests + i18n | 8 | camilogzlez |
| marketing | #13 | Pitch Deck investisseurs | 8 | Marinegyt |

**Total itération 4.3** : ~24 SP

---

## Jalons

| Date | Jalon | Statut |
|------|-------|--------|
| 17/11/2025 | Infrastructure opérationnelle | ✅ Atteint |
| 17/02/2026 | Moteur IA & génération | ✅ Atteint |
| 17/05/2026 | Application complète | ⏳ En cours (80%) |
| **17/08/2026** | **Application en production** | 🔮 Planifié |

---

## Charge par personne — PI 4

| Membre | SP planifiés PI 4 | Rôle PI 4 |
|--------|:-----------------:|-----------|
| **Marinegyt** | 34 | Marketing go-to-market, ASO, docs |
| **camilogzlez** | 32 | Web portal phases 3-6 |
| **FlorianBernier** | 21 | Tests payment + production |
| **vicous6** | 18 | Gateway rate-limit, logging, tests |
| **VictorVattierEpitech** | 13 | K8s deploy + tests GPU |
| **Romain-Ber** | 8 | Tests AI analysis |

> **Note** : Camserho (content-ingestion 100%) et TasseritNicolas (infra CI 100%) ont terminé leurs tâches. Disponibles en renfort.

---

## Risques identifiés

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Intégration Stripe E2E (3 services) | Retard PI 4.1 | Backend payment à 65%, réduire scope au checkout simple |
| Tests IA subjectifs (qualité images) | Pas de métrique claire | Définir critères objectifs (benchmark dataset) |
| Web portal à 49% d'avancement | Retard global | Focus camilogzlez sur phases 3-4 en priorité |

---

## Liens

- [Roadmap Globale](https://github.com/orgs/VisioBook-ESP/projects/5)
- [Marketing Kanban](https://github.com/orgs/VisioBook-ESP/projects/15)
- [Rapport Story Points](./story-points-report.md)
- [Diagramme de Gantt](./gantt.html)
- [User Stories](./user_stories.md)

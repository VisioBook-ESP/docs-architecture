# Rapport Story Points — VisioBook

> Dernière mise à jour : 2026-04-23

## Table de correspondance Story Points → Temps de développement

| Story Points | Effort estimé | Heures | Justification |
|:---:|:---:|:---:|:---|
| **1** | ¼ jour | ~2h | Fix mineur, config simple |
| **2** | ½ jour | ~4h | Petite tâche ciblée |
| **3** | 1 jour | ~7h | Tâche unitaire (endpoint, chart, CI) |
| **5** | 2 jours | ~14h | Module complet, intégration |
| **8** | 4 jours | ~28h | Phase complète, feature majeure |
| **13** | 7 jours | ~49h | Épique multi-composants |

**Base de calcul** : 1 SP ≈ 3.5h — vélocité mesurée sur le projet depuis août 2025.

> **Note méthodologique** : En Agile, les Story Points mesurent la complexité relative, pas le temps. Cette table de correspondance est une approximation basée sur notre vélocité observée, utilisée pour le reporting et la planification capacitaire.

---

## Répartition par membre de l'équipe

| Membre | Rôle principal | SP réalisés | SP restants | SP total | Jours réalisés | Jours restants |
|--------|---------------|:-----------:|:-----------:|:--------:|:--------------:|:--------------:|
| **vicous6** | Infra / DevOps / Gateway | 168 | 56 | 224 | 73j | 23j |
| **Marinegyt** | Mobile Flutter / Core User | 151 | 76 | 227 | 66j | 33j |
| **VictorVattierEpitech** | IA (Analysis + Media Gen) | 48 | 26 | 74 | 21j | 11j |
| **Romain-Ber** | Backend (Project + Analysis) | 44 | 16 | 60 | 19j | 7j |
| **Camserho** | Backend (Storage + Ingestion) | 44 | 5 | 49 | 19j | 2j |
| **FlorianBernier** | Backend (Payment) | 34 | 21 | 55 | 15j | 9j |
| **camilogzlez** | Frontend Web | 10 | 32 | 42 | 4j | 14j |
| **TasseritNicolas** | Infra (CI/CD) | 19 | 0 | 19 | 8j | 0j |
| **TOTAL** | | **518** | **232** | **750** | **225j** | **99j** |

---

## Répartition par service

| Service | SP Total | SP Done | SP Open | Jours total | Avancement |
|---------|:--------:|:-------:|:-------:|:-----------:|:----------:|
| Mobile Flutter App | 160 | 99 | 61 | 70j | 62% |
| Infra Helm Charts | 155 | 126 | 29 | 68j | 81% |
| Core User Service | 120 | 89 | 31 | 52j | 74% |
| AI Analysis Service | 65 | 47 | 18 | 28j | 72% |
| Core Payment Service | 61 | 40 | 21 | 27j | 66% |
| Content Ingestion Service | 47 | 47 | 0 | 20j | 100% |
| Support Storage Service | 47 | 37 | 10 | 20j | 79% |
| Web User Portal | 47 | 15 | 32 | 20j | 32% |
| AI Media Generation | 45 | 24 | 21 | 20j | 53% |
| Core API Gateway | 42 | 24 | 18 | 18j | 57% |
| Core Project Service | 39 | 31 | 8 | 17j | 79% |

---

## Métriques globales

| Métrique | Valeur |
|----------|--------|
| **Total Story Points** | 750 SP |
| **Story Points livrés** | 518 SP (69%) |
| **Story Points restants** | 232 SP (31%) |
| **Jours-homme réalisés** | ~225 jours |
| **Jours-homme restants** | ~99 jours |
| **Nombre d'issues** | 193 |
| **Issues fermées** | 146 (76%) |
| **Issues ouvertes** | 47 (24%) |
| **Membres actifs** | 8 |
| **Repos** | 11 microservices |

---

## Liens

- [Roadmap Globale (GitHub Project)](https://github.com/orgs/VisioBook-ESP/projects/5)
- [Diagramme de Gantt](./gantt.html)
- [User Stories](./user_stories.md)
- [Epics & Features](./epics.md)

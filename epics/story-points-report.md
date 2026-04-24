# Rapport Story Points — VisioBook

> Dernière mise à jour : 2026-04-24

## Table de correspondance Story Points → Temps de développement

| Story Points | Effort estimé | Heures |
|:---:|:---:|:---:|
| **1** | ~30 min | 0.5h |
| **2** | ~1h | 1h |
| **3** | ~1h30 | 1.5h |
| **5** | ~2h30 | 2.5h |
| **8** | ~3h30 (½ journée) | 3.5h |
| **13** | ~6h (~1 journée) | 6h |

**Base de calcul** : 1 SP ≈ 24 min — vélocité mesurée sur le projet.

**Contexte** : L'équipe travaille en moyenne 1 jour par semaine sur le projet. La correspondance SP → heures est calibrée sur la vélocité réelle observée depuis le début du projet (août 2025).

> **Note méthodologique** : En Agile, les Story Points mesurent la complexité relative, pas le temps. Cette table est une approximation basée sur notre vélocité observée, utilisée pour le reporting et la planification capacitaire.

---

## Répartition par membre de l'équipe

| Membre | Rôle principal | SP réalisés | SP restants | SP total | Heures réalisées | Jours réalisés |
|--------|---------------|:-----------:|:-----------:|:--------:|:----------------:|:--------------:|
| **vicous6** | Infra / DevOps / Gateway | 168 | 80 | 248 | 67h | 9.6j |
| **Marinegyt** | Mobile Flutter / Core User | 151 | 97 | 248 | 60h | 8.6j |
| **Romain-Ber** | Backend (Project + Analysis) | 148 | 24 | 172 | 59h | 8.5j |
| **VictorVattierEpitech** | IA (Analysis + Media + Storyboard) | 48 | 68 | 116 | 19h | 2.7j |
| **FlorianBernier** | Backend (Payment + Notification) | 34 | 50 | 84 | 14h | 1.9j |
| **Camserho** | Backend (Storage + Ingestion) | 44 | 5 | 49 | 18h | 2.5j |
| **camilogzlez** | Frontend Web | 10 | 45 | 55 | 4h | 0.6j |
| **TasseritNicolas** | Infra (CI/CD) | 19 | 0 | 19 | 8h | 1.1j |
| **TOTAL** | | **622** | **369** | **991** | **249h** | **35.5j** |

---

## Répartition par service

| Service | SP Total | SP Done | SP Open | Heures total | Avancement |
|---------|:--------:|:-------:|:-------:|:------------:|:----------:|
| Mobile Flutter App | 168 | 107 | 61 | 67h | 64% |
| Infra Helm Charts | 179 | 126 | 53 | 72h | 70% |
| Core Project Service | 143 | 135 | 8 | 57h | 94% |
| Core User Service | 120 | 89 | 31 | 48h | 74% |
| AI Analysis Service | 73 | 47 | 26 | 29h | 64% |
| AI Media Generation | 53 | 24 | 29 | 21h | 45% |
| Core Payment Service | 69 | 40 | 29 | 28h | 58% |
| Content Ingestion Service | 47 | 47 | 0 | 19h | 100% |
| Support Storage Service | 47 | 37 | 10 | 19h | 79% |
| Web User Portal | 60 | 15 | 45 | 24h | 25% |
| Core API Gateway | 42 | 24 | 18 | 17h | 57% |
| AI Storyboard Assembly | 34 | 0 | 34 | 14h | 0% |
| Core Notification Service | 21 | 0 | 21 | 8h | 0% |
| Docs / Transversal | 13 | 0 | 13 | 5h | 0% |

---

## Répartition par type (Enablers vs Features)

| Type | Issues | SP | % effort |
|------|:------:|:--:|:--------:|
| **enabler** | ~120 | ~400 | 40% |
| **feature** | ~85 | ~470 | 47% |
| **test** | ~18 | ~100 | 10% |
| **fix** / **deployment** | ~6 | ~21 | 3% |

---

## Métriques globales

| Métrique | Valeur |
|----------|--------|
| **Total Story Points** | 991 SP |
| **Story Points livrés** | 622 SP (63%) |
| **Story Points restants** | 369 SP (37%) |
| **Heures de dev réalisées** | ~249h |
| **Jours-homme réalisés** | ~35.5 jours |
| **Nombre d'issues** | 229 |
| **Issues fermées** | 164 (72%) |
| **Issues ouvertes** | 65 (28%) |
| **Membres actifs** | 8 |
| **Microservices** | 13 |
| **Durée du projet** | Août 2025 → Août 2026 (12 mois) |
| **Rythme** | ~1 jour/semaine par développeur |

---

## Liens

- [Roadmap Globale (GitHub Project)](https://github.com/orgs/VisioBook-ESP/projects/5)
- [Planning PI 3.3 & PI 4](./planning-pi3-pi4.md)
- [Diagramme de Gantt](./gantt.html)
- [User Stories](./user_stories.md)
- [Epics & Features](./epics.md)

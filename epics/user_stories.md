# User Stories du Projet Visiobook

## Vue d'ensemble
Ce document présente les User Stories du projet Visiobook, organisées par Milestone et Feature. Chaque User Story est classifiée selon la méthode MoSCoW (Must, Should, Could, Won't) pour indiquer sa priorité et est attribuée à une ou plusieurs équipes.

## Milestone 1: Importer un contenu

### US 1.1 – Import de fichiers
**MoSCoW**: Must
**Équipes**: Frontend, Backend, IA
**Feature associée**: 2.1 (Ingestion de Contenu)

**User Story**:
En tant qu'utilisateur, je veux pouvoir importer un fichier texte (PDF, TXT) pour lancer la génération de ma bande dessinée.

**Critères d'acceptation (BDD)**:
Given que je suis sur l'écran d'accueil,
When je sélectionne un fichier depuis mon téléphone,
Then l'application analyse automatiquement le contenu.

### US 1.2 – Scan de texte
**MoSCoW**: Should
**Équipes**: Frontend, IA
**Feature associée**: 2.1 (Ingestion de Contenu)

**User Story**:
En tant qu'utilisateur, je veux pouvoir scanner un texte avec mon téléphone pour générer une animation à partir de contenu papier.

**Critères d'acceptation (BDD)**:
Given que je n'ai pas de fichier numérique,
When je prends une photo d'un texte,
Then le texte est extrait et affiché dans un aperçu.

### US 1.3 – Prévisualisation et résumé
**MoSCoW**: Should
**Équipes**: Frontend, IA
**Feature associée**: 2.4 (Résumés et Condensation)

**User Story**:
En tant qu'utilisateur, je veux visualiser un résumé du contenu avant génération pour vérifier qu'il a été bien compris.

**Critères d'acceptation (BDD)**:
Given qu'un texte a été chargé,
When je clique sur « Aperçu »,
Then un résumé des chapitres ou scènes clés est affiché.

### US 1.4 – Validation du contenu
**MoSCoW**: Should
**Équipes**: Frontend, IA
**Feature associée**: 2.2 (Analyse Sémantique et Compréhension)

**User Story**:
En tant qu'utilisateur, je veux pouvoir valider ou modifier l'analyse du contenu pour m'assurer que l'IA a bien compris le texte.

**Critères d'acceptation (BDD)**:
Given que l'analyse du texte est affichée,
When je modifie certains éléments d'interprétation,
Then l'analyse est mise à jour en conséquence.

### US 1.5 – Extraction des scènes clés
**MoSCoW**: Must
**Équipes**: IA
**Feature associée**: 2.3 (Extraction de Scènes)

**User Story**:
En tant qu'utilisateur, je veux que le système identifie automatiquement les scènes clés de mon texte pour créer un storyboard cohérent.

**Critères d'acceptation (BDD)**:
Given qu'un texte a été analysé,
When le système traite le contenu,
Then les scènes importantes sont identifiées et structurées en storyboard.

## Milestone 2: Personnaliser le style de l'animation

### US 2.1 – Choix du style graphique
**MoSCoW**: Must
**Équipes**: Frontend, IA
**Feature associée**: 3.1 (Génération d'Images Statiques), 4.4 (Personnalisation et Paramètres Utilisateur)

**User Story**:
En tant qu'utilisateur, je veux choisir un style graphique (réaliste, cartoon, etc.) pour que l'animation corresponde à mes goûts.

**Critères d'acceptation (BDD)**:
Given qu'un contenu est prêt à être animé,
When je sélectionne un style dans la liste,
Then les scènes sont générées avec ce style.

### US 2.2 – Choix de la langue audio
**MoSCoW**: Should
**Équipes**: Frontend, IA
**Feature associée**: 3.3 (Synthèse Vocale et Dialogues), 4.4 (Personnalisation et Paramètres Utilisateur)

**User Story**:
En tant qu'utilisateur, je veux choisir la langue de la narration pour comprendre l'histoire dans ma langue.

**Critères d'acceptation (BDD)**:
Given que je configure mon projet,
When je sélectionne une langue,
Then la voix off et les dialogues sont adaptés à cette langue.

### US 2.3 – Définir la durée de l'animation
**MoSCoW**: Could
**Équipes**: Frontend, IA
**Feature associée**: 3.5 (Assemblage du Storyboard Final), 4.4 (Personnalisation et Paramètres Utilisateur)

**User Story**:
En tant qu'utilisateur, je veux ajuster la durée de la vidéo finale pour qu'elle s'adapte à mon temps disponible.

**Critères d'acceptation (BDD)**:
Given que j'ai lancé une génération,
When je sélectionne une durée courte ou longue,
Then le moteur adapte le niveau de détail dans la narration.

### US 2.4 – Personnalisation des personnages
**MoSCoW**: Could
**Équipes**: Frontend, IA
**Feature associée**: 3.1 (Génération d'Images Statiques), 4.4 (Personnalisation et Paramètres Utilisateur)

**User Story**:
En tant qu'utilisateur, je veux pouvoir personnaliser l'apparence des personnages principaux pour qu'ils correspondent à ma vision.

**Critères d'acceptation (BDD)**:
Given que des personnages ont été détectés dans mon texte,
When je modifie les attributs d'un personnage (âge, genre, style),
Then les images générées reflètent ces modifications.

### US 2.5 – Choix de l'ambiance sonore
**MoSCoW**: Could
**Équipes**: Frontend, IA
**Feature associée**: 3.4 (Effets Sonores et Musique), 4.4 (Personnalisation et Paramètres Utilisateur)

**User Story**:
En tant qu'utilisateur, je veux choisir le style musical et l'ambiance sonore pour renforcer l'atmosphère de mon histoire.

**Critères d'acceptation (BDD)**:
Given que je configure mon projet,
When je sélectionne un style musical ou une ambiance,
Then la bande sonore générée correspond à ce style.

## Milestone 3: Générer et visualiser une animation

### US 3.1 – Génération automatique
**MoSCoW**: Must
**Équipes**: IA, Backend
**Feature associée**: 3.1 (Génération d'Images Statiques), 3.2 (Création d'Animations), 3.5 (Assemblage du Storyboard Final)

**User Story**:
En tant qu'utilisateur, je veux que le système génère automatiquement une animation à partir de mon texte.

**Critères d'acceptation (BDD)**:
Given que j'ai configuré mon projet,
When je clique sur « Générer »,
Then une animation complète est produite avec son et images.

### US 3.2 – Visualisation dans l'application
**MoSCoW**: Must
**Équipes**: Frontend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux pouvoir lire l'animation générée dans l'application.

**Critères d'acceptation (BDD)**:
Given que la vidéo a été générée,
When j'ouvre le lecteur,
Then l'animation se lance avec son et image.

### US 3.3 – Suivi de la progression
**MoSCoW**: Should
**Équipes**: Frontend, Backend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux suivre la progression de la génération de mon animation pour savoir quand elle sera prête.

**Critères d'acceptation (BDD)**:
Given que j'ai lancé une génération,
When je consulte la page de statut,
Then je vois une barre de progression et une estimation du temps restant.

### US 3.4 – Prévisualisation des scènes
**MoSCoW**: Should
**Équipes**: Frontend, IA
**Feature associée**: 3.1 (Génération d'Images Statiques), 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux prévisualiser les scènes individuelles avant la génération complète pour valider la direction artistique.

**Critères d'acceptation (BDD)**:
Given que l'analyse du texte est terminée,
When je clique sur "Prévisualiser les scènes",
Then je vois un aperçu des images clés qui seront générées.

### US 3.5 – Contrôles de lecture
**MoSCoW**: Must
**Équipes**: Frontend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux disposer de contrôles de lecture standard pour naviguer dans mon animation.

**Critères d'acceptation (BDD)**:
Given que je visualise une animation,
When j'utilise les contrôles de lecture (play, pause, avance rapide),
Then l'animation répond correctement à ces commandes.

## Milestone 4: Exporter et partager l'animation

### US 4.1 – Télécharger la vidéo
**MoSCoW**: Must
**Équipes**: Frontend, Backend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux pouvoir télécharger la bande dessinée animée pour la conserver.

**Critères d'acceptation (BDD)**:
Given que l'animation est prête,
When je clique sur « Télécharger »,
Then la vidéo est enregistrée sur mon appareil.

### US 4.2 – Partage sur les réseaux
**MoSCoW**: Could
**Équipes**: Frontend, Backend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux pouvoir partager mon animation sur les réseaux sociaux depuis l'application.

**Critères d'acceptation (BDD)**:
Given que j'ai terminé une animation,
When je clique sur « Partager »,
Then je peux choisir une plateforme pour la publier.

### US 4.3 – Choix du format d'export
**MoSCoW**: Should
**Équipes**: Backend, IA
**Feature associée**: 3.5 (Assemblage du Storyboard Final)

**User Story**:
En tant qu'utilisateur, je veux choisir le format d'export de mon animation pour l'adapter à mes besoins.

**Critères d'acceptation (BDD)**:
Given que mon animation est prête,
When je sélectionne un format d'export (MP4, GIF, etc.),
Then l'animation est convertie dans ce format.

### US 4.4 – Génération de lien de partage
**MoSCoW**: Should
**Équipes**: Backend, Frontend
**Feature associée**: 4.1 (Application Web/Site Web), 4.2 (Application Mobile)

**User Story**:
En tant qu'utilisateur, je veux générer un lien de partage pour mon animation afin de l'envoyer directement à mes contacts.

**Critères d'acceptation (BDD)**:
Given que mon animation est prête,
When je clique sur "Générer un lien",
Then un lien unique est créé et copié dans mon presse-papier.

### US 4.5 – Protection par mot de passe
**MoSCoW**: Could
**Équipes**: Backend, Frontend
**Feature associée**: 5.4 (Gestion des Droits et Licences)

**User Story**:
En tant qu'utilisateur, je veux protéger mon animation par un mot de passe pour contrôler qui peut y accéder.

**Critères d'acceptation (BDD)**:
Given que j'ai généré un lien de partage,
When j'active l'option "Protéger par mot de passe" et que je définis un mot de passe,
Then l'accès à mon animation nécessite la saisie de ce mot de passe.

## Milestone 5: Historique et réutilisation

### US 5.1 – Accéder à l'historique de mes créations
**MoSCoW**: Should
**Équipes**: Frontend, Backend
**Feature associée**: 4.5 (Historique et Gestion des Projets)

**User Story**:
En tant qu'utilisateur, je veux pouvoir retrouver mes animations précédentes pour les revoir ou les modifier.

**Critères d'acceptation (BDD)**:
Given que j'ai déjà utilisé l'application,
When j'ouvre l'onglet « Mes projets »,
Then je vois la liste de mes créations passées.

### US 5.2 – Modifier un projet existant
**MoSCoW**: Could
**Équipes**: Frontend, Backend, IA
**Feature associée**: 4.5 (Historique et Gestion des Projets)

**User Story**:
En tant qu'utilisateur, je veux pouvoir ajuster un ancien projet sans tout recommencer.

**Critères d'acceptation (BDD)**:
Given qu'un projet est listé dans mon historique,
When je clique sur « Modifier »,
Then je peux changer le style, la langue ou la durée.

### US 5.3 – Dupliquer un projet
**MoSCoW**: Could
**Équipes**: Frontend, Backend
**Feature associée**: 4.5 (Historique et Gestion des Projets)

**User Story**:
En tant qu'utilisateur, je veux pouvoir dupliquer un projet existant pour créer une variante sans modifier l'original.

**Critères d'acceptation (BDD)**:
Given que je consulte un de mes projets,
When je clique sur "Dupliquer",
Then une copie du projet est créée avec un nouveau nom.

### US 5.4 – Organiser mes projets
**MoSCoW**: Could
**Équipes**: Frontend, Backend
**Feature associée**: 4.5 (Historique et Gestion des Projets)

**User Story**:
En tant qu'utilisateur, je veux pouvoir organiser mes projets en collections pour mieux les retrouver.

**Critères d'acceptation (BDD)**:
Given que j'ai plusieurs projets,
When je crée une collection et que j'y ajoute des projets,
Then mes projets sont organisés selon ces collections.

### US 5.5 – Supprimer un projet
**MoSCoW**: Should
**Équipes**: Frontend, Backend
**Feature associée**: 4.5 (Historique et Gestion des Projets)

**User Story**:
En tant qu'utilisateur, je veux pouvoir supprimer des projets que je ne souhaite plus conserver.

**Critères d'acceptation (BDD)**:
Given que je consulte mon historique,
When je sélectionne un projet et que je clique sur "Supprimer",
Then après confirmation, le projet est définitivement supprimé.

## User Stories supplémentaires par Epic

### Epic 1: Fondation et Infrastructure

#### US I1.1 – Surveillance des performances
**MoSCoW**: Should
**Équipes**: Infrastructure
**Feature associée**: 1.2 (Infrastructure Cloud et GPU)

**User Story**:
En tant qu'administrateur système, je veux surveiller les performances de l'infrastructure pour optimiser l'utilisation des ressources.

**Critères d'acceptation (BDD)**:
Given que l'infrastructure est en place,
When j'accède au tableau de bord de monitoring,
Then je peux voir les métriques de performance en temps réel.

#### US I1.2 – Gestion des sauvegardes
**MoSCoW**: Must
**Équipes**: Infrastructure, Backend
**Feature associée**: 1.3 (Base de Données et Stockage)

**User Story**:
En tant qu'administrateur système, je veux mettre en place un système de sauvegarde automatique pour protéger les données des utilisateurs.

**Critères d'acceptation (BDD)**:
Given que la base de données est en production,
When une sauvegarde est programmée,
Then les données sont sauvegardées de manière sécurisée et peuvent être restaurées si nécessaire.

### Epic 5: Système de Paiement et Gestion des Comptes

#### US P5.1 – Création de compte
**MoSCoW**: Must
**Équipes**: Frontend, Backend
**Feature associée**: 5.1 (Inscription et Authentification)

**User Story**:
En tant que nouvel utilisateur, je veux créer un compte pour accéder aux fonctionnalités de Visiobook.

**Critères d'acceptation (BDD)**:
Given que je suis sur la page d'accueil,
When je clique sur "S'inscrire" et que je remplis le formulaire,
Then mon compte est créé et je suis connecté.

#### US P5.2 – Abonnement premium
**MoSCoW**: Should
**Équipes**: Frontend, Backend
**Feature associée**: 5.2 (Gestion des Abonnements), 5.3 (Système de Paiement)

**User Story**:
En tant qu'utilisateur, je veux souscrire à un abonnement premium pour accéder à des fonctionnalités avancées.

**Critères d'acceptation (BDD)**:
Given que je suis connecté à mon compte,
When je choisis un plan d'abonnement et que je complète le paiement,
Then mon compte est mis à niveau et j'ai accès aux fonctionnalités premium.

### Epic 6: Qualité et Optimisation

#### US Q6.1 – Signalement de bugs
**MoSCoW**: Should
**Équipes**: Frontend, Testeur
**Feature associée**: 6.1 (Tests Fonctionnels et d'Intégration)

**User Story**:
En tant qu'utilisateur, je veux pouvoir signaler un bug ou un problème pour contribuer à l'amélioration de l'application.

**Critères d'acceptation (BDD)**:
Given que je rencontre un problème dans l'application,
When je clique sur "Signaler un problème" et que je décris le bug,
Then mon rapport est envoyé à l'équipe de développement.

#### US Q6.2 – Évaluation de la qualité IA
**MoSCoW**: Must
**Équipes**: IA, Testeur
**Feature associée**: 6.2 (Tests Spécifiques IA)

**User Story**:
En tant que testeur IA, je veux évaluer la qualité des générations pour améliorer les modèles.

**Critères d'acceptation (BDD)**:
Given qu'une animation a été générée,
When j'évalue la qualité selon plusieurs critères (cohérence, esthétique, fidélité au texte),
Then ces données sont utilisées pour améliorer les modèles d'IA.

### Epic 7: Déploiement et Maintenance

#### US D7.1 – Mise à jour des modèles IA
**MoSCoW**: Should
**Équipes**: IA, Infrastructure
**Feature associée**: 7.3 (Maintenance Évolutive)

**User Story**:
En tant qu'administrateur, je veux pouvoir mettre à jour les modèles d'IA sans interruption de service.

**Critères d'acceptation (BDD)**:
Given qu'une nouvelle version d'un modèle d'IA est disponible,
When je lance le processus de mise à jour,
Then le modèle est mis à jour sans impact sur les utilisateurs actifs.

#### US D7.2 – Analyse des retours utilisateurs
**MoSCoW**: Should
**Équipes**: Testeur, Frontend
**Feature associée**: 7.2 (Monitoring et Support)

**User Story**:
En tant que responsable produit, je veux analyser les retours des utilisateurs pour prioriser les améliorations.

**Critères d'acceptation (BDD)**:
Given que des retours utilisateurs ont été collectés,
When j'accède au tableau de bord d'analyse,
Then je peux voir les tendances et les problèmes les plus fréquemment signalés.

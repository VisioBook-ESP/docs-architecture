# RAPPORT D'ÉTUDE DE FAISABILITÉ LÉGALE
## PROJET VISIOBOOK

---

### ÉQUIPE : BETTER CALL SAUL
### DATE : 19 septembre 2025
### VERSION : 1.0

---

## 1. DESCRIPTION DU PROJET VISIOBOOK

### 1.1 Présentation générale

VisioBook est une application innovante qui transforme des livres en expériences audiovisuelles interactives à l'aide de l'intelligence artificielle. Le projet vise à révolutionner la consommation de contenu littéraire en créant des adaptations multimédia automatisées.

### 1.2 Fonctionnalités principales

**Ingestion de contenu :**
- Import de fichiers texte (PDF, TXT)
- Scan et reconnaissance optique de caractères (OCR)
- Prétraitement et analyse sémantique du contenu

**Génération automatique :**
- Extraction automatique des scènes clés
- Génération d'images statiques et d'animations
- Synthèse vocale et création de dialogues
- Génération d'effets sonores et de musique
- Assemblage en storyboard final

**Services utilisateur :**
- Interface web et application mobile
- Système de personnalisation (style graphique, langue, durée)
- Gestion des projets et historique
- Fonctionnalités d'export et de partage

### 1.3 Architecture technique

Le système repose sur une architecture microservices dans le cloud avec :
- Infrastructure GPU pour les modèles d'IA
- Base de données pour la gestion des utilisateurs et projets
- Système de stockage pour les contenus générés
- APIs pour l'intégration des services tiers

### 1.4 Modèle économique

- Modèle freemium avec abonnements premium
- Système de paiement intégré
- Monétisation basée sur l'usage et les fonctionnalités avancées

---

## 2. ANALYSE COMPARATIVE DES ASPECTS LÉGAUX

### 2.1 Propriété intellectuelle et droits d'auteur

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Transformation d'œuvres protégées par la communauté<br>• Contributions tierces non contrôlées<br>• Responsabilité partagée difficile à établir<br>• Usage commercial par des tiers du code | • Transformation automatique d'œuvres protégées<br>• Utilisation commerciale d'œuvres sans autorisation<br>• Responsabilité éditeur directe<br>• Conflits droits moraux des auteurs |
| **Articles/lois** | • Code PI (L. 122-4, L. 335-2)<br>• Directive 2001/29/CE<br>• Licences open source (Apache 2.0) | • Code PI (L. 122-4, L. 335-2)<br>• Directive 2001/29/CE<br>• Directive 2012/28/UE œuvres orphelines |
| **Impact potentiel** | • Procédures contre contributeurs individuels<br>• Retrait forcé du code problématique<br>• Fragmentation du projet | • Injonctions cessation commerciale<br>• Dommages jusqu'à 300k€/œuvre<br>• Blocage développement commercial |
| **Stratégies préventives** | • CLA (Contributor License Agreement)<br>• Audit communautaire du contenu<br>• Domaine public exclusif initial<br>• Gouvernance claire contributions | • Partenariats maisons d'édition<br>• Focus domaine public strict<br>• Veille juridique automatisée<br>• Base données autorisations |
| **Stratégies techniques** | • Fork autorisé du code propre<br>• Détection automatique œuvres protégées<br>• Validation communautaire | • API vérification droits temps réel<br>• Workflow validation juridique<br>• Traçabilité complète sources |
| **Stratégies curatives** | • Take-down communautaire<br>• Médiation bénévole<br>• Exclusion contributeurs récidivistes | • Take-down rapide (<24h)<br>• Médiation professionnelle<br>• Fonds indemnisation ayants droit |
| **Coûts potentiels** | • 15k-25k€ (conseil juridique licence + CLA) | • 50k-150k€ (licences + conseil spécialisé) |

### 2.2 Protection des données personnelles (RGPD/CNIL)

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Données personnelles limitées (pas de paiement)<br>• Contributions communautaires tracées<br>• Transferts Azure pendant développement<br>• **Futur :** Analytics communautaires | • Données identification + paiement complètes<br>• Contenus personnels uploadés<br>• Transferts internationaux Azure<br>• **Futur :** Analytics comportementales |
| **Articles/lois** | • RGPD articles 6, 7, 17, 20<br>• Loi Informatique et Libertés<br>• Guidelines EDPB contributions | • RGPD articles 6, 7, 17, 20, 22<br>• Loi Informatique et Libertés<br>• Guidelines EDPB consentement |
| **Impact potentiel** | • Sanctions limitées (CA=0)<br>• Responsabilité partagée communauté<br>• Notifications violations 72h | • Sanctions 4% CA mondial ou 20M€<br>• Responsabilité éditeur complète<br>• Actions collectives possibles |
| **Stratégies préventives** | • Privacy by design simplifié<br>• DPO bénévole/externe ponctuel<br>• Minimisation données contributeurs<br>• AIPD allégée | • Privacy by design complet<br>• DPO interne temps partiel<br>• AIPD systématique tous traitements<br>• Minimisation stricte données |
| **Stratégies techniques** | • Pseudonymisation contributeurs<br>• Chiffrement basique (AES-256)<br>• Hébergement UE après migration<br>• API RGPD simplifiée | • Chiffrement end-to-end complet<br>• Pseudonymisation automatique<br>• Migration Azure → UE obligatoire<br>• API RGPD complète automatisée |
| **Stratégies curatives** | • Effacement contributions possibles<br>• Portabilité code source<br>• Plan réponse incidents communautaire | • Effacement créations techniques<br>• Portabilité BD séquencée JSON<br>• Audit sécurité trimestriel externe |
| **Coûts potentiels** | • 10k-15k€ (DPO externe ponctuel + audit)<br>• Migration Azure → UE : 15k-25k€ | • 25k-35k€ (DPO temps partiel + audit)<br>• Migration Azure → UE : 15k-25k€ |

### 2.3 Accessibilité numérique (RGAA)

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Interface communautaire non accessible<br>• Contributions sans descriptions alt<br>• Navigation non optimisée lecteurs d'écran | • Interface commerciale non accessible<br>• BD générées sans descriptions alt<br>• Navigation séquentielle non optimisée |
| **Articles/lois** | • RGAA 4.1 (pas obligatoire startup)<br>• Loi 2005-102 égalité droits<br>• Responsabilité sociale | • RGAA 4.1 (pas obligatoire startup)<br>• Loi 2005-102 égalité droits<br>• **Évolution possible vers obligation** |
| **Impact potentiel** | • Exclusion contributeurs handicapés<br>• Image négative communauté<br>• Adoption limitée | • Exclusion 12M personnes (France)<br>• Perte marché significative<br>• Atteinte image marque commercial |
| **Stratégies préventives** | • Développement a11y communautaire<br>• Formation bénévoles RGAA<br>• Tests avec associations | • Développement a11y first professionnel<br>• Formation équipe complète RGAA<br>• Tests utilisateurs systématiques |
| **Stratégies techniques** | • Générateur alt-text IA basique<br>• Navigation clavier communautaire<br>• Mode contraste simple | • Générateur alt-text IA avancé<br>• Navigation clavier complète BD<br>• Mode contraste + agrandissement |
| **Coûts potentiels** | • 3k-5k€ (formation communauté + tests) | • 8k-12k€ (formation équipe + tests utilisateurs) |

### 2.4 Règlement européen sur l'IA (AI Act)

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Système IA risque limité (génération contenu)<br>• Transparence algorithmes communautaires<br>• Documentation partagée requise<br>• Supervision communautaire difficile | • Système IA risque limité (génération commerciale)<br>• Transparence algorithmes commerciaux<br>• Documentation technique complète requise<br>• Supervision humaine professionnelle |
| **Articles/lois** | • AI Act articles 50-55 (risque limité)<br>• Guidelines CNIL IA open source<br>• Responsabilité partagée communauté | • AI Act articles 50-55 (risque limité)<br>• Guidelines CNIL IA commerciale<br>• Responsabilité éditeur directe |
| **Impact potentiel** | • Amendes 1,5% CA (CA=0 donc limitées)<br>• Retrait du code problématique<br>• Fragmentation projet si non-compliance | • Amendes 1,5% CA mondial<br>• Retrait du marché obligatoire<br>• Responsabilité contenus générés |
| **Stratégies préventives** | • Documentation modèles ouverte<br>• Comité éthique IA communautaire<br>• Tests biais collaboratifs<br>• Gouvernance transparente | • Documentation modèles complète<br>• Comité éthique IA interne<br>• Tests biais automatisés systématiques<br>• Certification professionnelle |
| **Stratégies techniques** | • Logs décisions IA publics<br>• Supervision communautaire<br>• Filtres pré-génération basiques<br>• Watermarking open source | • Logs décisions IA sécurisés<br>• Interface supervision pro avec alertes<br>• Filtres pré-génération avancés<br>• Watermarking invisible propriétaire |
| **Stratégies curatives** | • Intervention communautaire<br>• Audit externe bénévole<br>• Mise à jour modèles communautaire | • Intervention humaine rapide (<1h)<br>• Audit externe certifié annuel<br>• Plan mise à jour modèles pro |
| **Coûts potentiels** | • 10k-20k€ (documentation + audit externe) | • 25k-40k€ (audit + certification + documentation) |

### 2.5 Aspects commerciaux et financiers

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • **Aucun paiement** = pas de risques DSP2<br>• Pas de rétractation (service gratuit)<br>• CGU associatives simples<br>• Obligations fiscales nulles (CA=0) | • Non-conformité DSP2 authentification<br>• Violation droit rétractation 14 jours<br>• Clauses contractuelles abusives<br>• Obligations fiscales complexes (TVA numérique) |
| **Articles/lois** | • **Non applicable** (pas de commerce)<br>• Droit des associations<br>• Déclarations préfecture | • Directive DSP2 (2015/2366)<br>• Code consommation (L. 221-18)<br>• Directive 2011/83/UE consommateurs |
| **Impact potentiel** | • **Risques nuls** (pas de commerce)<br>• Responsabilité associative limitée<br>• Pas de sanctions pécuniaires | • Sanctions 50k€ (DSP2)<br>• Remboursements forcés<br>• Redressements fiscaux + pénalités |
| **Stratégies préventives** | • **Non applicable** (pas de paiements)<br>• Statuts associatifs conformes<br>• Déclarations administratives à jour | • PSP agréé DSP2 (Stripe/PayPal)<br>• CGV/CGU juriste e-commerce<br>• Procédure rétractation automatisée |
| **Stratégies techniques** | • **Non applicable** (pas de paiements)<br>• Site web informatif simple<br>• Système donations optionnel | • API paiement 3D Secure 2.0<br>• Facturation automatique conforme<br>• Dashboard gestion remboursements |
| **Stratégies curatives** | • **Non applicable** (pas de commerce)<br>• Assurance associative basique<br>• Conseil juridique ponctuel | • Assurance RC pro 2M€ minimum<br>• Médiation consommation obligatoire<br>• Conseil fiscal spécialisé international |
| **Coûts potentiels** | • **0€** (pas d'obligations commerciales)<br>• Statuts association : 1k-2k€ | • 15k-25k€ (juridique + assurance + fiscal) |

### 2.6 Responsabilité et contenu généré

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Contenu offensant généré par communauté<br>• Contributions inappropriées non contrôlées<br>• Statut hébergeur communautaire<br>• Responsabilité partagée difficile | • Contenu offensant généré commercialement<br>• Reproduction involontaire éléments protégés<br>• Statut éditeur de contenu<br>• Responsabilité directe pour tous contenus |
| **Articles/lois** | • LCEN 2004-575 (hébergeur)<br>• Digital Services Act (plateforme)<br>• Responsabilité collective limitée | • LCEN 2004-575 (éditeur/hébergeur)<br>• Digital Services Act (plateforme commerciale)<br>• Code civil 1240-1245 (RC complète) |
| **Impact potentiel** | • Actions RC limitées (CA=0)<br>• Modération communautaire<br>• Fragmentation si problèmes | • Actions RC pour préjudice moral<br>• Obligations modération lourdes<br>• Coûts modération humaine élevés |
| **Stratégies préventives** | • Modération collaborative basique<br>• CGU communautaires simples<br>• Signalement bénévole | • Modération automatisée professionnelle<br>• CGU strictes usage commercial<br>• Signalement utilisateur intégré |
| **Stratégies techniques** | • Filtres basiques open source<br>• Blacklist communautaire<br>• Validation collaborative<br>• Watermarking visible | • IA détection contenu pro (MS Moderator)<br>• Blacklist évolutive avancée<br>• Validation humaine systématique<br>• Watermarking invisible + traçabilité |
| **Stratégies curatives** | • Take-down communautaire<br>• Modération bénévole<br>• Exclusion contributeurs | • Take-down automatisée (<2h)<br>• Équipe modération pro (2 ETP min)<br>• Service modération externe |
| **Coûts potentiels** | • 5k-15k€ (outils modération basiques) | • 60k-80k€ (service + équipe modération) |

### 2.7 Distribution sur les app stores (Play Store/App Store)

| **Critère** | **Modèle Open Source Gratuit** | **Modèle Commercial Payant** |
|-------------|--------------------------------|------------------------------|
| **Risques identifiés** | • Conformité politiques stores (app gratuite)<br>• Classification PEGI pour contenu IA<br>• **Pas de DMA** (pas de revenus)<br>• Contenu accessible mineurs | • Conformité politiques stores strictes<br>• Classification PEGI obligatoire payante<br>• **DMA applicable** si gros revenus<br>• Responsabilité mineurs + monétisation |
| **Articles/lois** | • Politiques App Store/Play (gratuites)<br>• Code industrie jeu (PEGI)<br>• **DMA non applicable** (CA=0) | • Politiques développeur commerciales<br>• DMA 2022/1925 (gatekeepers)<br>• DSA 2022/2065 (plateformes payantes) |
| **Impact potentiel** | • Refus publication (95% marché mobile)<br>• Restriction âge limitant adoption<br>• **Pas de commission** (app gratuite) | • Refus/suppression stores (95% marché)<br>• Sanctions DMA 10% CA mondial<br>• **Commission 15-30% revenus** |
| **Stratégies préventives** | • Audit politiques stores gratuits<br>• Classification 12+ conservatrice<br>• Documentation process IA<br>• **Pas de certification payante** | • Audit politiques stores commerciales<br>• Classification 12+ minimum<br>• Documentation complète review<br>• Certification PEGI/ESRB professionnelle |
| **Stratégies techniques** | • Filtrage basique contenus mineurs<br>• Contrôle parental simple<br>• Métadonnées basiques<br>• APIs officielles respectées | • Filtrage avancé contenus mineurs<br>• Contrôle parental intégré complet<br>• Métadonnées détaillées algorithmes<br>• Conformité APIs complète |
| **Stratégies curatives** | • Procédure appel gratuite<br>• Version unique<br>• Support communautaire stores | • Procédure appel structurée<br>• Versions régionales multiples<br>• Support développeur dédié stores |
| **Coûts potentiels** | • 3k-8k€ (classification PEGI + audit basic)<br>• **0% commission** (app gratuite) | • 10k-15k€ (PEGI/ESRB + audit complet)<br>• **15-30% commission** sur revenus mobile |

---

## 3. RECOMMANDATIONS ET CONCLUSION

### 3.1 Faisabilité légale du projet

Le projet VisioBook est **légalement réalisable** dans les deux modèles économiques envisagés, sous réserve de la mise en œuvre rigoureuse des stratégies de mitigation identifiées.

#### 3.1.1 Modèle Open Source Gratuit
**Investissement juridique :** 150 000 - 250 000 €
**Avantages :** Responsabilité limitée, communauté d'audit, financement public possible
**Inconvénients :** Contrôle limité de l'usage, contributions tierces à gérer

#### 3.1.2 Modèle Commercial Premium
**Investissement juridique :** 220 000 - 380 000 €
**Avantages :** Contrôle total, ressources pour compliance, modèle économique viable
**Inconvénients :** Responsabilité éditeur complète, obligations commerciales lourdes

### 3.2 Contraintes techniques critiques

**Hébergement Azure obligatoire :**
- Impact RGPD : Transferts de données vers les États-Unis pendant le développement
- Stratégie de migration post-développement vers hébergeur UE recommandée
- Coût de migration : 15 000 - 25 000 €

**Distribution mobile prioritaire :**
- 95% du marché accessible via App Store et Play Store
- Classification d'âge obligatoire (12+ recommandé)
- Commission 15-30% sur revenus + coûts certification 10 000 - 15 000 €

### 3.3 Priorités d'implémentation révisées

**Phase 1 (Critique - 0-3 mois) :**
- **Choix du modèle économique** (open source vs commercial)
- Focus exclusif sur œuvres domaine public (risque PI minimal)
- Mise en conformité RGPD avec contrainte Azure
- Classification d'âge et audit politiques des stores
- Rédaction CGU/CGV adaptées au modèle choisi

**Phase 2 (Important - 3-6 mois) :**
- Documentation technique AI Act (logs, traçabilité, supervision)
- Systèmes de modération renforcés (contenus accessibles aux mineurs)
- Développement accessible (RGAA) pour avantage concurrentiel
- Préparation migration hébergement (si modèle commercial)

**Phase 3 (Souhaitable - 6-8 mois) :**
- Certifications externes (PEGI/ESRB, audit IA)
- Migration effective vers hébergement UE (si applicable)
- Négociations licences éditoriales pour expansion du catalogue
- Publication sur les app stores

### 3.4 Analyse coût/bénéfice comparative

#### Modèle Open Source
**Coûts totaux :** 150 000 - 250 000 €
**Revenus :** Donations, subventions publiques, sponsoring
**Risque juridique :** Faible à modéré
**Contrôle :** Limité (gouvernance communautaire)

#### Modèle Commercial
**Coûts totaux :** 220 000 - 380 000 € + commissions stores (15-30%)
**Revenus :** Abonnements, achats in-app, licences
**Risque juridique :** Modéré à élevé
**Contrôle :** Total

### 3.5 Facteurs de décision pour le choix du modèle

**Opter pour l'Open Source si :**
- Budget limité (< 250 000 €)
- Objectif éducatif/social prioritaire
- Équipe expérimentée en gouvernance communautaire
- Financement public/associatif disponible

**Opter pour le Commercial si :**
- Budget suffisant (> 300 000 €)
- Objectif de rentabilité à moyen terme
- Contrôle total du produit requis
- Ressources humaines pour la compliance disponibles

### 3.6 Recommandations finales

#### Pour le modèle Open Source
**APPROBATION conditionnée à :**
- Budget minimal de 200 000 € (marge sécurité 30%)
- Structure associative à but non lucratif
- Licence Apache 2.0 avec CLA
- DPO interne temps partiel
- Planning de 6 mois de préparation légale

#### Pour le modèle Commercial
**APPROBATION conditionnée à :**
- Budget minimal de 350 000 € (marge sécurité 30%)
- Société commerciale avec juriste spécialisé permanent
- Assurance RC professionnelle 2M€ minimum
- DPO interne temps plein ou externe
- Planning de 8 mois de préparation légale

### 3.7 Points d'attention critiques (tous modèles)

1. **Stratégie droits d'auteur :** Succès conditionné au focus initial domaine public
2. **Contrainte Azure :** Migration UE obligatoire avant commercialisation (modèle payant)
3. **App stores :** 95% du marché mobile, compliance critique
4. **Classification d'âge :** Impact direct sur l'audience cible
5. **Modération IA :** Investissement lourd mais obligatoire pour crédibilité

**Conclusion :** Les deux modèles sont viables juridiquement. Le choix dépend de la stratégie commerciale, des ressources disponibles et de la tolérance au risque de l'équipe.

---

## SOURCES LÉGALES ET RÉGLEMENTAIRES

### Propriété intellectuelle
- Code de la propriété intellectuelle (L. 122-4, L. 335-2)
- Directive 2001/29/CE sur l'harmonisation des droits d'auteur
- Directive 2012/28/UE sur les œuvres orphelines
- Jurisprudence CJUE C-5/08 Infopaq

### Protection des données
- Règlement (UE) 2016/679 (RGPD)
- Loi n° 78-17 du 6 janvier 1978 modifiée (Loi Informatique et Libertés)
- Délibération CNIL n° 2022-046 du 26 mai 2022
- Guidelines EDPB 05/2019 sur le consentement

### Accessibilité numérique
- Référentiel Général d'Amélioration de l'Accessibilité (RGAA 4.1)
- Directive (UE) 2016/2102 relative à l'accessibilité
- Loi n° 2005-102 du 11 février 2005 pour l'égalité des droits et des chances
- Décret n° 2019-768 du 24 juillet 2019

### Intelligence artificielle
- Règlement (UE) 2024/1689 (AI Act)
- Recommandations UNESCO sur l'éthique de l'IA (2021)
- Guidelines CNIL sur l'IA et protection des données
- Livre blanc CE sur l'IA (2020)

### Droit commercial et financier
- Directive (UE) 2015/2366 (DSP2)
- Code de la consommation (L. 221-18 et suivants)
- Directive 2011/83/UE relative aux droits des consommateurs
- Code général des impôts (TVA numérique)

### Responsabilité et contenus
- Loi n° 2004-575 du 21 juin 2004 (LCEN)
- Directive 2000/31/CE sur le commerce électronique
- Règlement (UE) 2022/2065 (Digital Services Act)
- Code civil (articles 1240-1245 - responsabilité civile)

---

**Fin du rapport - 5 pages**

# Bounded Contexts — Carte des services

> Chaque bounded context est un service autonome avec son propre modèle de données et son langage ubiquitaire.
> Les services communiquent via des événements domaine — jamais via accès direct à la base de données d'un autre service.

---

## Vue d'ensemble

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  catalogue  │     │   content   │     │   search    │
│             │     │             │     │             │
│ Taxonomie   │     │ Guides      │     │ Full-text   │
│ Fiches      │     │ Articles    │     │ Filtres     │
│ Espèces     │     │ Médias      │     │ Suggestions │
└─────────────┘     └─────────────┘     └─────────────┘

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  identity   │     │  collection │     │  social     │
│             │     │             │     │             │
│ Auth        │     │ Spécimens   │     │ Profils     │
│ Profils     │     │ Carnet      │     │ Communauté  │
│ Grades      │     │ Rappels     │     │ Échanges    │
└─────────────┘     └─────────────┘     └─────────────┘

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│identification│    │gamification │     │notification │
│             │     │             │     │             │
│ Analyse     │     │ Points      │     │ Push        │
│ AR          │     │ Badges      │     │ Email       │
│ Historique  │     │ Défis       │     │ In-app      │
└─────────────┘     └─────────────┘     └─────────────┘

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  geo        │     │ marketplace │     │    i18n     │
│             │     │             │     │             │
│ Carte       │     │ Annonces    │     │ Interface   │
│ Événements  │     │ Échanges    │     │ Traductions │
│ Biodiversité│     │ Transactions│     │ Communauté  │
└─────────────┘     └─────────────┘     └─────────────┘
```

---

## Gateway — Point d'entrée unique

La gateway est le seul point de contact entre les clients (mobile, web) et les services internes.

```
Client mobile / web
        │
        ▼
   ┌─────────┐
   │ Gateway │  ← authentification, routing, rate limiting, aggregation
   └─────────┘
        │
   ┌────┴────┬────────┬────────┬────────┐
   ▼         ▼        ▼        ▼        ▼
catalogue  identity  social   geo    ...autres
```

**Responsabilités** :
- Authentification et vérification des tokens (JWT)
- Routing des requêtes vers le bon service
- Rate limiting par utilisateur
- Agrégation de réponses multi-services (BFF — Backend For Frontend)
- Gestion des erreurs centralisée
- Logging et tracing des requêtes

**Ce qu'elle ne fait pas** :
- Aucune logique métier
- Aucun accès direct aux bases de données des services
- Aucun stockage de données métier

---

## Détail des services

### catalogue
**Responsabilité** : référentiel botanique de toutes les espèces
- Familles, genres, espèces, cultivars, hybrides
- Données d'entretien, morphologie, caractéristiques
- Source de vérité pour tout ce qui concerne une espèce végétale
- Contenu validé par des experts

**Entités principales** : Family, Genus, Species, Plant (fiche)
**Événements émis** : PlantCreated, PlantPublished, PlantUpdated

---

### collection
**Responsabilité** : gestion des spécimens possédés par un utilisateur
- Ajout / suppression de spécimens dans une collection
- Carnet de bord (journal d'entretien)
- Rappels et alertes d'entretien
- Calcul de l'âge d'un spécimen

**Entités principales** : Collection, Specimen, JournalEntry, Reminder
**Événements émis** : SpecimenAdded, JournalEntryCreated, ReminderTriggered
**Dépend de** : catalogue (pour les données de l'espèce), identity (utilisateur)

---

### identity
**Responsabilité** : authentification et profil utilisateur
- Inscription, connexion, OAuth (Google, Apple)
- Profil public (pseudo, avatar, bio, localisation par ville)
- Type de profil (débutant, passionné, jardinier, expert, pro)
- Préférences et paramètres

**Entités principales** : User, Profile, UserPreferences
**Événements émis** : UserRegistered, ProfileUpdated, ProfileTypeChanged

---

### gamification
**Responsabilité** : système de progression et récompenses
- Calcul et attribution des points
- Gestion des grades et badges
- Défis (individuels, collectifs, saisonniers)
- Saisons de jeu

**Entités principales** : UserProgress, Badge, Challenge, Season
**Événements émis** : PointsEarned, BadgeUnlocked, GradeReached, ChallengeCompleted
**Dépend de** : identity (utilisateur), écoute tous les événements domaine pour attribuer des points

---

### identification
**Responsabilité** : identification des plantes par photo ou AR
- Analyse d'image (via API externe ou modèle interne)
- Historique des identifications d'un utilisateur
- Diagnostic de maladies et carences
- AR temps réel

**Entités principales** : Identification, DiagnosticResult
**Événements émis** : PlantIdentified, DiagnosticCompleted
**Dépend de** : catalogue (matching avec les espèces), identity (utilisateur)

---

### social
**Responsabilité** : interactions entre utilisateurs
- Questions / réponses
- Commentaires et likes
- Système expert / débutant
- Messagerie privée
- Groupes thématiques
- Live streaming

**Entités principales** : Post, Comment, Message, Group, ExpertProfile
**Événements émis** : PostCreated, ExpertAnswered, GroupCreated
**Dépend de** : identity (utilisateur), catalogue (références aux espèces)

---

### geo
**Responsabilité** : tout ce qui est géographique et cartographique
- Carte des observations communautaires
- Événements à proximité
- Parcs et jardins botaniques
- Zones climatiques
- Données sol par région

**Entités principales** : Observation, CommunityEvent, BotanicGarden, ClimateZone
**Événements émis** : ObservationSubmitted, EventCreated
**Dépend de** : catalogue (espèces observées), identity (utilisateur)

---

### notification
**Responsabilité** : envoi de toutes les notifications
- Push mobile, email, in-app
- Agrégation intelligente (pas de spam)
- Préférences de notification par utilisateur
- Templates de messages

**Entités principales** : Notification, NotificationTemplate, UserPreferences
**Écoute** : événements de tous les autres services
**N'émet pas** d'événements métier

---

### search
**Responsabilité** : recherche full-text et suggestions
- Recherche de plantes, guides, utilisateurs, événements
- Suggestions et autocomplétion
- Filtres avancés
- Moteur : Meilisearch

**Dépend de** : catalogue, social, geo (indexation des contenus)
**N'émet pas** d'événements métier

---

### marketplace
**Responsabilité** : échanges économiques entre utilisateurs
- Annonces de vente, échange, don de plantes et boutures
- Transactions et paiements
- Notation des vendeurs
- Boutique graines rares

**Entités principales** : Listing, Transaction, Review
**Événements émis** : ListingCreated, TransactionCompleted
**Dépend de** : identity (utilisateur), catalogue (espèces)

---

### content
**Responsabilité** : contenu éditorial et communautaire
- Guides et articles experts
- Fiches enrichies (vidéos, podcasts)
- Contenu généré par les utilisateurs (UGC) avec modération
- Traductions communautaires

**Entités principales** : Article, Guide, Media, Translation
**Événements émis** : ArticlePublished, ContentValidated
**Dépend de** : catalogue (espèces référencées), identity (auteurs)

---

### media
**Responsabilité** : gestion centralisée de tous les médias
- Upload, stockage et optimisation des images
- Transcoding des vidéos
- Hébergement des modèles 3D
- Génération de thumbnails et variantes (mobile, web, AR)
- CDN et distribution

**Entités principales** : Media, MediaVariant, UploadJob
**Événements émis** : MediaUploaded, MediaProcessed, MediaDeleted
**Consommé par** : tous les services qui affichent des médias

---

### analytics
**Responsabilité** : statistiques personnelles et insights
- Calcul des stats de collection (taux de survie, régularité...)
- Génération du rapport annuel "Folia Wrapped"
- Prédictions et projections
- Agrégation des données pour les insights communautaires

**Entités principales** : UserStats, CollectionInsight, AnnualReport
**Événements émis** : AnnualReportGenerated
**Écoute** : tous les événements domaine pour construire les métriques
**Ne stocke pas** de données métier — agrège uniquement

---

### i18n
**Responsabilité** : gestion centralisée des traductions de l'interface et contributions communautaires

**Deux périmètres distincts** :

**1. Traductions interface (i18n technique)**
- Labels, messages, notifications, erreurs de l'application
- Géré via des fichiers de traduction (JSON/YAML) par langue
- Langues prioritaires au lancement : français, anglais, espagnol, allemand, portugais
- Outil de gestion : Crowdin ou équivalent

**2. Traductions de données métier (contributions communautaires)**
- Noms vernaculaires des espèces (dans `catalogue`)
- Articles et guides (dans `content`)
- Ces traductions restent dans leurs services respectifs — `i18n` ne les duplique pas
- `i18n` fournit uniquement les outils de contribution et de validation

**Entités principales** : TranslationKey, TranslationContribution, Language, TranslationReview
**Événements émis** : TranslationContributed, TranslationValidated, LanguageActivated
**Consommé par** : tous les services pour l'interface, `catalogue` et `content` pour les données métier

**Langues et statuts** :

| Statut | Signification |
|---|---|
| `ACTIVE` | Langue complète, disponible dans l'app |
| `PARTIAL` | En cours de traduction, accessible en beta |
| `COMMUNITY` | Traduction communautaire, pas encore validée |
| `INACTIVE` | Langue désactivée |

**Règles** :
- Une langue n'est activée que si elle atteint 90% de couverture des clés critiques
- Les traductions communautaires sont validées par deux locuteurs natifs minimum avant publication
- Le français et l'anglais sont les langues de référence — toute nouvelle clé doit d'abord exister dans ces deux langues

---

## Types de communication entre services

### Synchrone (REST / gRPC)
Utilisé quand une réponse immédiate est nécessaire.
- `gateway → catalogue` : récupérer une fiche plante
- `gateway → identity` : vérifier un token
- `identification → catalogue` : matcher une espèce identifiée

### Asynchrone (événements / message broker)
Utilisé pour découpler les services et garantir la résilience.
- `collection → gamification` : SpecimenAdded → attribuer des points
- `identification → geo` : PlantIdentified → créer une observation
- Tous les événements vers `notification`
- Tous les événements vers `analytics`

### Règle générale
> Un service ne fait **jamais** de requête synchrone vers un autre service pour des opérations non critiques. Si la réponse n'est pas nécessaire immédiatement, on utilise des événements.

---

## Ordre de développement suggéré

| Phase | Services | Raison |
|---|---|---|
| MVP | catalogue, identity, collection, notification, i18n | Base indispensable |
| V1 étendu | identification, gamification, search, media | Différenciants |
| V2 | social, geo, content, analytics | Engagement et communauté |
| V3 | marketplace | Monétisation |
| V4 | audit | Sécurité avancée et modération |

---

## Services futurs — Notes

### audit *(prévu V4)*
**Responsabilité** : traçabilité, détection de fraude et modération centralisée
- Audit log de toutes les actions sensibles (connexions, suppressions, modifications critiques)
- Détection de comportements suspects (spam, abus de gamification, faux comptes)
- Tableau de bord de modération pour l'équipe Folia
- Centralisation des signalements de contenu et d'utilisateurs

> La sécurité de base (auth, JWT, rate limiting, RBAC) est gérée par `identity` et la `gateway`.
> Le service `audit` n'est nécessaire qu'à partir du moment où la plateforme a suffisamment d'utilisateurs pour justifier une modération active.

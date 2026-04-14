# Agrégats DDD — Par service

> Un agrégat est une unité de cohérence transactionnelle.
> Tout ce qui doit rester cohérent ensemble forme un agrégat.
> On ne modifie jamais les entités internes d'un agrégat directement — on passe toujours par la racine.

## Règles de suppression globales

| Type | Stratégie | Délai de purge |
|---|---|---|
| **Hard delete** | Suppression immédiate et définitive | Immédiat |
| **Soft delete** | Marqué `deletedAt`, invisible mais conservé | Configurable |
| **Anonymisation** | Données personnelles remplacées, contenu conservé | Sur demande RGPD |

**Par agrégat** :

| Agrégat | Stratégie | Raison |
|---|---|---|
| `Plant` (catalogue) | Soft delete | Des spécimens peuvent y faire référence |
| `Species` | Soft delete | Vérification des dépendances requise |
| `Genus` | Soft delete | Vérification des dépendances requise |
| `Family` | Soft delete | Vérification des dépendances requise |
| `Specimen` | Soft delete 90 jours puis hard delete | Conserver l'historique temporairement |
| `JournalEntry` | Jamais supprimé (immuable) | Traçabilité de l'entretien |
| `Reminder` | Hard delete | Pas de valeur historique |
| `User` | Anonymisation (RGPD) sous 30 jours | Obligation légale |
| `Identification` | Hard delete après 30 jours si non convertie | Images sensibles |
| `Observation` | Soft delete | Contribution biodiversité à préserver |
| `Post` | Soft delete | Conserver les réponses associées |
| `Listing` | Soft delete 1 an | Historique des transactions |
| `Transaction` | Jamais supprimé | Obligation légale (comptabilité) |

---

## catalogue

### Plant (racine d'agrégat)
**Entités internes** : aucune (les données taxonomiques sont des références)
**Value objects** : ScientificName, Care, Characteristics, Appearance, PlantId
**Invariants** :
- Le nom scientifique doit être unique dans le catalogue
- Une plante doit obligatoirement référencer une espèce existante
- Une plante ne peut être publiée que si elle a un nom scientifique, une espèce et des données d'entretien complètes
- Une plante supprimée ne peut plus être publiée

**Événements émis** : PlantCreated, PlantPublished, PlantUnpublished, PlantUpdated, PlantDeleted

---

### Species (racine d'agrégat)
**Entités internes** : aucune
**Value objects** : ScientificName, TaxonomyReference
**Invariants** :
- Une espèce doit appartenir à un genre existant
- Le nom scientifique doit être unique
- Une espèce ne peut pas être supprimée si des plantes y font référence

**Événements émis** : SpeciesCreated, SpeciesUpdated

---

### Genus (racine d'agrégat)
**Entités internes** : aucune
**Invariants** :
- Un genre doit appartenir à une famille existante
- Le nom latin doit être unique
- Un genre ne peut pas être supprimé s'il contient des espèces

**Événements émis** : GenusCreated, GenusUpdated

---

### Family (racine d'agrégat)
**Entités internes** : aucune
**Invariants** :
- Le nom latin doit être unique
- Une famille ne peut pas être supprimée si elle contient des genres

**Événements émis** : FamilyCreated, FamilyUpdated

---

## collection

### Specimen (racine d'agrégat)
**Entités internes** : JournalEntry (liste)
**Value objects** : PlantingDate, SpecimenAge, SpecimenId
**Invariants** :
- Un spécimen doit référencer une espèce existante dans le catalogue
- Un spécimen appartient à un seul utilisateur
- La date de plantation ne peut pas être dans le futur
- Les entrées du journal sont immuables une fois créées
- Un spécimen supprimé conserve son journal (soft delete)

**Événements émis** : SpecimenAdded, SpecimenRemoved, JournalEntryCreated, CareStreakUpdated

---

### Reminder (racine d'agrégat)
**Entités internes** : aucune
**Value objects** : ReminderSchedule, ReminderType
**Invariants** :
- Un rappel doit être associé à un spécimen existant
- La date de rappel doit être dans le futur lors de la création
- Un rappel complété ne peut pas être re-déclenché (créer un nouveau)

**Événements émis** : ReminderTriggered, ReminderCompleted

---

## identity

### User (racine d'agrégat)
**Entités internes** : Profile
**Value objects** : Email, UserId, ProfileType, CityLocation
**Invariants** :
- L'email doit être unique sur la plateforme
- Le pseudo doit être unique et ne contenir que des caractères alphanumériques
- La localisation est stockée au niveau ville/département — jamais GPS précis
- Un utilisateur supprimé (RGPD) voit toutes ses données personnelles effacées mais ses contributions publiques anonymisées

**Événements émis** : UserRegistered, UserVerified, ProfileUpdated, ProfileTypeChanged, UserDeleted

---

## identification

### Identification (racine d'agrégat)
**Entités internes** : aucune
**Value objects** : ConfidenceScore, IdentificationMethod, ImageReference
**Invariants** :
- Une identification doit avoir au moins une image
- Le score de confiance est entre 0 et 100
- Une identification ne peut pas être modifiée après création (immuable)
- En dessous de 60% de confiance, une validation humaine est recommandée

**Événements émis** : PlantIdentified, IdentificationFailed

---

### Diagnostic (racine d'agrégat)
**Entités internes** : DiagnosticResult (liste de causes probables)
**Value objects** : ConfidenceScore, Symptom
**Invariants** :
- Un diagnostic est toujours lié à un spécimen ou une image
- Les résultats sont classés par probabilité décroissante
- Un diagnostic est immuable une fois généré

**Événements émis** : DiagnosticCompleted

---

## gamification

### UserProgress (racine d'agrégat)
**Entités internes** : BadgeCollection, ChallengeProgress
**Value objects** : Grade, Points, Streak
**Invariants** :
- Les points ne peuvent jamais être négatifs
- La progression de grade est uniquement croissante (on ne rétrograde pas)
- Un badge obtenu ne peut pas être retiré (sauf abus)
- Le streak se remet à zéro si aucune action n'est effectuée pendant 24h

**Événements émis** : PointsEarned, BadgeUnlocked, GradeReached, StreakBroken

---

### Challenge (racine d'agrégat)
**Entités internes** : ChallengeObjective (liste)
**Value objects** : ChallengeType, ChallengePeriod, Reward
**Invariants** :
- Un défi a toujours une date de début et une date de fin
- Un défi expiré ne peut plus être complété
- Les objectifs d'un défi ne changent pas après publication

**Événements émis** : ChallengeCreated, ChallengeCompleted, SeasonEnded

---

## social

### Post (racine d'agrégat)
**Entités internes** : Comment (liste)
**Value objects** : PostContent, PostId
**Invariants** :
- Un post ne peut pas être vide
- Un post supprimé conserve les réponses mais affiche "contenu supprimé"
- Un expert peut marquer une réponse comme "validée" — une seule réponse validée par post

**Événements émis** : PostCreated, PostLiked, CommentAdded, ExpertAnswered

---

## geo

### Observation (racine d'agrégat)
**Entités internes** : aucune
**Value objects** : Region, ObservationDate, ConfidenceScore
**Invariants** :
- Une observation doit référencer une espèce identifiée
- La localisation est stockée au niveau ville/commune — jamais GPS précis
- Une observation validée ne peut plus être modifiée

**Événements émis** : ObservationSubmitted, ObservationValidated

---

## marketplace

### Listing (racine d'agrégat)
**Entités internes** : aucune
**Value objects** : ListingPrice, ListingType (vente, échange, don), ListingStatus
**Invariants** :
- Une annonce doit référencer une espèce du catalogue
- Une annonce clôturée ne peut plus recevoir de demandes
- Le vendeur ne peut pas transacter avec lui-même
- Une transaction ne peut être initiée que si l'annonce est active

**Événements émis** : ListingCreated, ListingClosed, TransactionCompleted

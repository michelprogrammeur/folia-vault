# Langage Ubiquitaire par Contexte

> Le même mot peut avoir un sens différent selon le service dans lequel il est utilisé.
> Ce fichier documente ces nuances pour éviter toute confusion dans le code et les discussions.

---

## "Plant" / Plante

| Service | Signification | Nom technique |
|---|---|---|
| `catalogue` | Fiche de référence d'une espèce végétale | `Plant` (aggregate) |
| `collection` | Exemplaire physique possédé par un utilisateur | `Specimen` |
| `identification` | Résultat d'une analyse — espèce probable | `IdentifiedSpecies` |
| `marketplace` | Objet d'une annonce de vente ou d'échange | `ListingItem` |
| `geo` | Espèce observée dans la nature | `ObservedSpecies` |

> ⚠️ Dans le code, on ne dit jamais "plant" dans le service `collection` — on dit toujours "specimen".

---

## "User" / Utilisateur

| Service | Signification | Nom technique |
|---|---|---|
| `identity` | Compte avec credentials, email, mot de passe | `User` |
| `social` | Profil public visible par les autres | `Profile` |
| `gamification` | Joueur avec points, grade, badges | `Player` |
| `collection` | Propriétaire d'une collection de spécimens | `Owner` |
| `marketplace` | Acheteur ou vendeur dans une transaction | `Buyer` / `Seller` |

---

## "Event" / Événement

| Service | Signification | Nom technique |
|---|---|---|
| `collection` | Action enregistrée dans le carnet d'un spécimen | `JournalEntry` |
| `geo` | Sortie, conférence, marché organisé | `CommunityEvent` |
| `messaging` (interne) | Message entre services du système | `DomainEvent` |
| `gamification` | Action déclenchant l'attribution de points | `GameAction` |

---

## "Observation"

| Service | Signification | Nom technique |
|---|---|---|
| `identification` | Acte d'observer et photographier une plante | `Identification` |
| `geo` | Observation validée contribuée à la biodiversité | `Observation` |
| `social` | Commentaire ou note partagé avec la communauté | `Post` |

---

## "Collection"

| Service | Signification | Nom technique |
|---|---|---|
| `collection` | Ensemble des spécimens d'un utilisateur | `Collection` |
| `gamification` | Ensemble des cartes botaniques débloquées | `CardCollection` |
| `catalogue` | Ensemble des fiches plantes du catalogue | (pas d'agrégat — c'est une query) |

---

## "Notification"

| Service | Signification | Nom technique |
|---|---|---|
| `collection` | Rappel lié à l'entretien d'un spécimen | `Reminder` |
| `notification` | Message envoyé à l'utilisateur (push, email) | `Notification` |
| `gamification` | Alerte de progression (nouveau badge, grade) | `ProgressAlert` |

---

## "Score"

| Service | Signification | Nom technique |
|---|---|---|
| `identification` | Niveau de confiance d'une identification (0-100%) | `ConfidenceScore` |
| `gamification` | Points accumulés par un utilisateur | `Points` |
| `marketplace` | Note laissée après une transaction (0-5) | `Rating` |
| `social` | Nombre de votes positifs sur un post | `LikeCount` |

---

## "Status" / Statut

| Contexte | Valeurs possibles |
|---|---|
| Spécimen (`collection`) | `ALIVE`, `DORMANT`, `DECEASED` |
| Plante catalogue (`catalogue`) | `DRAFT`, `PUBLISHED`, `UNPUBLISHED` |
| Annonce (`marketplace`) | `DRAFT`, `ACTIVE`, `RESERVED`, `CLOSED` |
| Utilisateur (`identity`) | `PENDING_VERIFICATION`, `ACTIVE`, `SUSPENDED`, `DELETED` |
| Article (`content`) | `DRAFT`, `PENDING_REVIEW`, `PUBLISHED`, `REJECTED` |
| Défi (`gamification`) | `UPCOMING`, `ACTIVE`, `COMPLETED`, `EXPIRED` |

---

## Règles de nommage dans le code

- Dans `catalogue` : on parle de `Plant`, jamais de `Specimen` ni de `PlantInstance`
- Dans `collection` : on parle de `Specimen`, jamais de `Plant` ni de `PlantInstance`
- Dans `identity` : on parle de `User` pour le compte, `Profile` pour la représentation publique
- Dans `gamification` : on parle de `Player` pour éviter la confusion avec `User`
- Les événements domaine sont toujours nommés au **passé** : `PlantCreated`, pas `CreatePlant`
- Les commands sont nommées à **l'infinitif** : `CreatePlant`, pas `PlantCreated`
- Les queries sont nommées avec **Get ou List** : `GetPlant`, `ListPlants`

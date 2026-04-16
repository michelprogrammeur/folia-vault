# ADR-008 — Stratégie de suppression : soft delete vs hard delete

## Statut
Accepté

## Contexte
Quand un utilisateur supprime une plante, faut-il la retirer définitivement de la base de données (hard delete) ou la marquer comme supprimée (soft delete) ?

## Options considérées

### Option 1 — Hard delete
Suppression physique de la ligne en base de données.
- ✅ Simple — pas de logique supplémentaire
- ✅ Pas de données "fantômes" en base
- ✅ RGPD : les données sont vraiment supprimées
- ❌ Irréversible — impossible de récupérer une plante supprimée par erreur
- ❌ Casse les foreign keys si d'autres services référencent cette plante
- ❌ Pas d'historique pour l'audit

### Option 2 — Soft delete ✅
Ajout d'un champ `deletedAt` — la ligne reste en base mais est filtrée.
- ✅ Réversible — une plante supprimée peut être restaurée
- ✅ Historique préservé pour l'audit
- ✅ Les foreign keys des autres services restent valides
- ✅ Compatible avec les domain events (`PlantDeleted` conservé en history)
- ❌ Complexité accrue — toutes les queries doivent filtrer `WHERE deletedAt IS NULL`
- ❌ RGPD : nécessite un mécanisme de purge périodique pour les vraies suppressions

### Option 3 — Soft delete avec purge planifiée
Soft delete immédiat + job de purge après 30 jours.
- ✅ Meilleur des deux mondes
- ✅ Conformité RGPD possible
- ❌ Complexité opérationnelle (job de purge)

## Décision
**Soft delete avec `deletedAt`.**

## Raisons
- Les plantes peuvent être référencées par d'autres services (collections, identifications...)
- La récupération après suppression accidentelle est une demande prévisible
- L'historique est précieux pour les analytics et l'audit
- Un hard delete cassant des références croisées est inacceptable dans un microservices

## Conséquences
- La colonne `deletedAt TIMESTAMP` est ajoutée à la table `plants`
- Toutes les queries de lecture filtrent `WHERE plants.deleted_at IS NULL`
- `DeletePlantHandler` met à jour `deletedAt = NOW()` au lieu de supprimer
- Un index partiel `WHERE deleted_at IS NULL` optimise les queries de lecture
- `PlantRepositoryPort` expose une méthode `softDelete(id: PlantId): Promise<void>`
- La purge RGPD est une tâche future (job planifié ou trigger après X jours)

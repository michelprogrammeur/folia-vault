# Stratégie de sauvegarde

## Principes
- **Règle 3-2-1** : 3 copies, 2 supports différents, 1 hors site
- Les sauvegardes sont testées régulièrement — une sauvegarde non testée n'existe pas
- RTO (Recovery Time Objective) : < 4 heures pour un P1
- RPO (Recovery Point Objective) : < 1 heure de perte de données acceptable

## Bases de données PostgreSQL

### Fréquence
- Sauvegarde complète : 1 fois par jour (nuit)
- WAL archiving (point-in-time recovery) : continu

### Rétention
| Type | Durée |
|------|-------|
| Sauvegardes journalières | 30 jours |
| Sauvegardes hebdomadaires | 3 mois |
| Sauvegardes mensuelles | 1 an |

### Stockage
- Sauvegarde primaire : même région cloud
- Sauvegarde secondaire : région cloud différente (ou S3-compatible)
- Chiffrement des sauvegardes au repos (AES-256)

### Outils
- `pg_dump` / `pg_basebackup` pour les sauvegardes
- WAL-G ou Barman pour le point-in-time recovery
- Les hébergeurs managés (Supabase, Neon, Railway) incluent les sauvegardes automatiques

## Fichiers utilisateurs (images)

### Fréquence
- Réplication synchrone vers un bucket S3 secondaire
- Les objets S3 ne sont pas "backupés" au sens classique — la réplication multi-région suffit

### Rétention
- Images actives : conservées tant que le compte existe
- Images supprimées : 30 jours (soft delete) puis purge

## Configuration et secrets

### Code
- Le code est dans Git — GitHub est la source de vérité
- Pas de sauvegarde supplémentaire nécessaire

### Secrets
- Secrets stockés dans un gestionnaire (Vault, Doppler, AWS Secrets Manager)
- Export chiffré des secrets : 1 fois par semaine
- Stockage offline dans un endroit sécurisé

## Tests de restauration
- **Mensuel** : test de restauration d'une base de données complète
- **Trimestriel** : test de restauration point-in-time
- Les tests sont documentés et les résultats archivés

## À faire
- [ ] Choisir l'hébergeur PostgreSQL (Supabase, Neon, Railway...)
- [ ] Configurer WAL archiving ou vérifier que l'hébergeur le propose
- [ ] Configurer la réplication S3 multi-région pour les images
- [ ] Planifier le premier test de restauration

# Runbooks

Procédures opérationnelles step-by-step. À consulter pendant un incident ou pour les opérations de maintenance courantes.

---

## Redémarrer un service

```bash
# Vérifier l'état du service
fly status --app folia-catalogue

# Redémarrer
fly restart --app folia-catalogue

# Vérifier les logs post-redémarrage
fly logs --app folia-catalogue
```

---

## Appliquer une migration en production

⚠️ Toujours faire une sauvegarde avant une migration.

```bash
# 1. Sauvegarde manuelle
pg_dump $DATABASE_URL > backup-$(date +%Y%m%d-%H%M%S).sql

# 2. Vérifier la migration en staging d'abord
pnpm --filter catalogue migrate:staging

# 3. Appliquer en production
pnpm --filter catalogue migrate:prod

# 4. Vérifier que le service répond correctement
curl https://api.folia.app/health
```

---

## Vider / débloquer une queue BullMQ

```bash
# Ouvrir Bull Board
open https://admin.folia.app/queues

# Ou via CLI
pnpm --filter catalogue queue:drain <queue-name>      # Vider tous les jobs en attente
pnpm --filter catalogue queue:retry-failed <queue-name> # Rejouer les jobs échoués
```

---

## Rollback d'un déploiement

```bash
# Lister les releases récentes
fly releases --app folia-catalogue

# Rollback vers la version précédente
fly deploy --image registry.fly.io/folia-catalogue:<sha-précédent>
```

---

## Investiguer une erreur 500 en production

```bash
# 1. Consulter les logs des dernières 30 minutes
fly logs --app folia-catalogue | grep '"level":50'  # level 50 = error Pino

# 2. Chercher l'erreur dans Grafana
# Dashboard : Folia / Errors → filtrer par service et route

# 3. Identifier le request ID dans les logs et tracer la requête
# Header : X-Request-ID
```

---

## Forcer la déconnexion d'un utilisateur (sécurité)

```sql
-- Révoquer tous les refresh tokens d'un utilisateur
UPDATE refresh_tokens
SET revoked_at = NOW()
WHERE user_id = '<user-id>';
```

---

## Purger les données d'un compte supprimé (RGPD)

```bash
# Lancer le job de purge manuellement pour un utilisateur
pnpm --filter identity purge:user <user-id>
```

---

## À compléter
- [ ] Ajouter les runbooks spécifiques à chaque service au fur et à mesure
- [ ] Lier les alertes Grafana aux runbooks correspondants

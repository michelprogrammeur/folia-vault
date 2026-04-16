# Cache — Stratégie

## Principe

Mettre en cache uniquement ce qui est mesuré comme lent ou coûteux.
Chaque couche de cache ajoute de la complexité — ne l'introduire que si nécessaire.

---

## Couches de cache

```
Client
  │ Cache HTTP (Cache-Control, ETag)
  ▼
CDN (médias, pages statiques)
  │
  ▼
Gateway
  │ Cache applicatif (Redis)
  ▼
Service
  │ Cache requêtes DB (Redis)
  ▼
PostgreSQL
```

---

## Cache HTTP

### Pages statiques (Astro SSG)
- Fiches plantes générées statiquement → servis par le CDN
- `Cache-Control: public, max-age=3600, stale-while-revalidate=86400`
- Invalidation via revalidation Astro à chaque publication

### API publique (catalogue)
```
Cache-Control: public, max-age=300    ← 5 minutes pour les listes
Cache-Control: public, max-age=3600   ← 1 heure pour une fiche
ETag: "hash-du-contenu"               ← validation conditionnelle
```

### API privée (collection, gamification...)
```
Cache-Control: private, no-store      ← jamais de cache pour les données utilisateur
```

---

## Redis — Cache applicatif

### Ce qu'on met en cache

| Données | TTL | Invalidation |
|---|---|---|
| Fiche plante complète | 1 heure | À la modification |
| Liste des familles/genres | 24 heures | À la modification |
| Profil utilisateur | 15 minutes | À la modification |
| Résultats de recherche fréquents | 10 minutes | Automatique (TTL) |
| Sessions utilisateur | 30 jours | À la déconnexion |
| Rate limiting counters | 1 minute / 1 jour | Automatique (TTL) |

### Ce qu'on ne met PAS en cache
- Données de collection (carnet, spécimens) — trop personnelles et changeantes
- Notifications — doivent être temps réel
- Données de transaction marketplace — cohérence critique

### Pattern Cache-Aside (standard utilisé)

```typescript
async function getPlant(id: string): Promise<Plant> {
  // 1. Vérifier le cache
  const cached = await redis.get(`plant:${id}`)
  if (cached) return JSON.parse(cached)

  // 2. Chercher en base
  const plant = await plantRepository.findById(id)
  if (!plant) throw new PlantNotFoundError(id)

  // 3. Mettre en cache
  await redis.setex(`plant:${id}`, 3600, JSON.stringify(plant))

  return plant
}
```

### Invalidation du cache

```typescript
// À chaque modification d'une plante
async function invalidatePlantCache(id: string): Promise<void> {
  await redis.del(`plant:${id}`)
  await redis.del('plants:list:*')   // invalider toutes les listes
}
```

### Nommage des clés Redis

Format : `<service>:<type>:<identifiant>`

```
catalogue:plant:uuid-123
catalogue:plants:list:page-1-limit-20
identity:user:uuid-456
identity:session:token-hash
```

---

## CDN — Médias

- Toutes les images et vidéos servis via CDN (Cloudflare R2 ou AWS CloudFront)
- URLs immuables — inclure le hash du contenu dans l'URL
- `Cache-Control: public, max-age=31536000, immutable` ← 1 an pour les médias

```
https://cdn.folia.app/plants/monstera-deliciosa-abc123.webp
```

- Si l'image change → nouvelle URL → pas d'invalidation nécessaire

---

## Cache Meilisearch

- L'index Meilisearch est lui-même un cache des données de recherche
- Mis à jour via les événements domaine (asynchrone)
- Pas de cache supplémentaire devant Meilisearch — il est déjà très rapide

---

## Monitoring du cache

- Taux de hit/miss Redis par clé (alerte si < 80% de hit sur les données chaudes)
- Mémoire utilisée par Redis (alerte si > 80% de la mémoire allouée)
- Latence Redis (alerte si > 10ms)
- Taille de l'index Meilisearch

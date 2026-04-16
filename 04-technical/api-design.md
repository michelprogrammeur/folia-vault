# API Design — Conventions REST

## Principes généraux

- Toutes les APIs sont RESTful et exposent du JSON
- Versioning via le préfixe de route : `/v1/plants`
- Les noms de routes sont en **kebab-case** et au **pluriel**
- Les actions non-CRUD utilisent des sous-ressources verbales

## Versioning

```
/v1/plants          → version 1
/v2/plants          → version 2 (si breaking change)
```

- La version courante est `v1`
- Une ancienne version est maintenue minimum 6 mois après la sortie d'une nouvelle
- Les breaking changes déclenchent obligatoirement un bump de version

## Nommage des routes

```
# Ressources — pluriel, kebab-case
GET    /v1/plants
GET    /v1/plants/:id
POST   /v1/plants
PATCH  /v1/plants/:id
DELETE /v1/plants/:id

# Sous-ressources
GET    /v1/plants/:id/journal-entries
POST   /v1/plants/:id/journal-entries

# Actions non-CRUD — verbes explicites
POST   /v1/plants/:id/publish
POST   /v1/plants/:id/unpublish
POST   /v1/auth/refresh-token
POST   /v1/auth/logout
```

## Méthodes HTTP et codes de retour

| Opération | Méthode | Succès | Erreur courante |
|---|---|---|---|
| Lister | `GET` | 200 | 400 (filtres invalides) |
| Récupérer | `GET` | 200 | 404 (non trouvé) |
| Créer | `POST` | 201 | 400 (validation), 409 (conflit) |
| Mettre à jour | `PATCH` | 204 | 400, 404 |
| Remplacer | `PUT` | 204 | 400, 404 |
| Supprimer | `DELETE` | 204 | 404 |
| Action | `POST` | 204 | 400, 404, 409 |

## Format des réponses

### Ressource unique
```json
{
  "id": "uuid",
  "scientificName": "Monstera deliciosa",
  "type": "PLANT",
  "createdAt": "2025-04-14T10:30:00Z",
  "updatedAt": "2025-04-14T10:30:00Z"
}
```

### Liste paginée
```json
{
  "data": [...],
  "total": 150,
  "page": 1,
  "limit": 20,
  "totalPages": 8
}
```

### Création réussie
```json
{
  "plantId": "uuid"
}
```

### Erreur
```json
{
  "error": "Plant not found: uuid",
  "code": "PLANT_NOT_FOUND",
  "statusCode": 404
}
```

### Erreur de validation
```json
{
  "error": "Validation error",
  "details": {
    "scientificName": ["Required"],
    "care.watering": ["Invalid enum value"]
  }
}
```

## Paramètres de requête

### Pagination
```
?page=1&limit=20
```

### Filtres
```
?type=PLANT&difficulty=EASY&isPublished=true
```

### Tri
```
?sortBy=createdAt&sortOrder=desc
```

### Recherche
```
?q=monstera
```

## Conventions de nommage des champs

- **camelCase** pour tous les champs JSON
- **ISO 8601 UTC** pour les dates : `"2025-04-14T10:30:00Z"`
- **UUID v4** pour tous les identifiants
- Booléens préfixés par `is` ou `has` : `isPublished`, `isPetFriendly`

## Headers obligatoires

### Requête
```
Content-Type: application/json
Authorization: Bearer <jwt_token>
```

### Réponse
```
Content-Type: application/json
X-Request-Id: uuid   ← corrélation avec les logs
```

## Idempotence

- `GET`, `PUT`, `DELETE` sont idempotents par nature
- `POST` de création n'est pas idempotent — utiliser une clé d'idempotence si nécessaire
- `PATCH` est idempotent si les mêmes données sont envoyées

## Règles à ne pas enfreindre

- Ne jamais retourner un tableau vide `[]` à la racine — toujours envelopper dans `{ data: [] }`
- Ne jamais exposer les IDs internes de la base de données — toujours des UUIDs
- Ne jamais retourner des données d'un autre service dans la réponse — chaque service retourne ses propres données
- Ne jamais ignorer silencieusement des champs inconnus dans le body — retourner une erreur 400

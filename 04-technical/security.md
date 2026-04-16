# Sécurité

## Authentification

### JWT (JSON Web Tokens)
- Tokens signés avec une clé secrète (HS256 minimum, RS256 en production)
- Access token : durée de vie courte (15 minutes)
- Refresh token : durée de vie longue (30 jours), stocké en base
- Rotation automatique des refresh tokens à chaque usage

### OAuth 2.0
- Connexion via Google et Apple (V1)
- Le service `identity` est le seul à gérer les tokens OAuth
- Les autres services reçoivent uniquement un `userId` validé

### Flux d'authentification
```
Client → Gateway → identity (vérification token)
                       ↓
                   userId validé
                       ↓
              Gateway → Service cible
```

## Autorisation — RBAC

### Rôles
| Rôle | Description |
|---|---|
| `USER` | Utilisateur standard |
| `EXPERT` | Utilisateur vérifié avec statut expert |
| `MODERATOR` | Peut modérer les contenus |
| `ADMIN` | Accès complet à la plateforme |

### Permissions par rôle
| Action | USER | EXPERT | MODERATOR | ADMIN |
|---|---|---|---|---|
| Lire le catalogue | ✅ | ✅ | ✅ | ✅ |
| Créer un spécimen | ✅ | ✅ | ✅ | ✅ |
| Valider une observation | ❌ | ✅ | ✅ | ✅ |
| Publier un article | ❌ | ✅ | ✅ | ✅ |
| Modérer un contenu | ❌ | ❌ | ✅ | ✅ |
| Créer une fiche catalogue | ❌ | ❌ | ❌ | ✅ |

## Rate Limiting

Géré au niveau de la gateway :

| Endpoint | Limite | Fenêtre |
|---|---|---|
| Identification (gratuit) | 5 req | par jour |
| Identification (premium) | illimité | — |
| Auth (login) | 10 req | par heure |
| API générale | 100 req | par minute |
| Upload médias | 20 req | par heure |

## Données personnelles — RGPD

- Localisation stockée au niveau ville uniquement — jamais GPS
- Données personnelles chiffrées au repos en production
- Droit à l'oubli : suppression ou anonymisation sous 30 jours
- Consentement explicite requis pour les données optionnelles
- Logs anonymisés — jamais d'email ou nom dans les logs
- Durées de rétention documentées par type de donnée

## Sécurité applicative

### Validation des entrées
- Toutes les entrées HTTP validées avec Zod avant traitement
- Pas de construction de requêtes SQL dynamiques (Drizzle ORM)
- Sanitisation des contenus texte avant stockage

### Headers de sécurité
- `Content-Security-Policy`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Strict-Transport-Security` (HTTPS obligatoire en production)

### Secrets
- Jamais de secrets dans le code ou les commits
- Variables d'environnement uniquement
- `.env` dans `.gitignore`
- En production : gestionnaire de secrets (Vault, AWS Secrets Manager...)

## 2FA *(V1 étendu)*
- TOTP (Google Authenticator, Authy)
- Obligatoire pour les comptes ADMIN et MODERATOR
- Optionnel pour les autres rôles

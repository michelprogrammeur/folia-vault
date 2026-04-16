# Politique de sécurité

## Principes
- **Defence in depth** : plusieurs couches de sécurité
- **Least privilege** : chaque service n'a accès qu'à ce dont il a besoin
- **Fail secure** : en cas de doute, refuser l'accès
- **Security by design** : la sécurité est intégrée dès la conception, pas ajoutée après

## Authentification & Autorisation

### Utilisateurs
- JWT (access token court — 15 min + refresh token long — 30 jours)
- Mots de passe hashés avec bcrypt (coût 12)
- Rate limiting sur les routes d'authentification (5 tentatives / 15 min)
- 2FA optionnel (TOTP) — obligatoire pour les comptes admin

### Services internes
- Communication inter-services via tokens signés (service-to-service auth)
- Réseau privé — les services ne sont pas exposés directement sur Internet
- Variables d'environnement pour les secrets — jamais dans le code

## Sécurité applicative (OWASP Top 10)

| Risque | Mitigation |
|--------|-----------|
| Injection SQL | Drizzle ORM avec requêtes paramétrées |
| XSS | Échappement automatique (React/Vue) + CSP headers |
| CSRF | Tokens CSRF + SameSite cookies |
| Broken Auth | JWT + refresh tokens + rate limiting |
| Sensitive Data | TLS 1.3 + chiffrement au repos |
| Security Misconfiguration | Headers HTTP sécurisés (`@fastify/helmet`) |
| Vulnerable Dependencies | `pnpm audit` en CI |
| SSRF | Validation des URLs externes avant requête |

## Infrastructure

### Réseau
- Services exposés uniquement via un reverse proxy / API gateway
- Firewall : seuls les ports 80/443 sont ouverts publiquement
- VPC privé pour la communication inter-services

### Secrets
- Jamais de secrets dans le code ou les variables d'environnement en clair en prod
- Gestionnaire de secrets : Doppler, HashiCorp Vault, ou AWS Secrets Manager
- Rotation des secrets tous les 90 jours

### Mises à jour
- Dépendances mises à jour chaque semaine (`pnpm audit`)
- Images Docker basées sur des images officielles minimales (Alpine)
- OS patché automatiquement par l'hébergeur (managed)

## Gestion des vulnérabilités

### Découverte
- `pnpm audit` en CI — build échoue si vulnérabilité critique
- Dependabot / Renovate pour les mises à jour automatiques
- Bug bounty à terme (HackerOne, YesWeHack)

### Signalement responsable
Contact sécurité : security@folia.app (à créer)
Délai de traitement : 90 jours avant divulgation publique

## Audit & Conformité
- Logs d'accès aux données sensibles (audit trail) — voir `gdpr.md`
- Revue de sécurité avant chaque mise en production majeure
- Pentest annuel à partir de la V2

## À faire
- [ ] Configurer `@fastify/helmet` sur tous les services
- [ ] Mettre en place le rate limiting sur les routes auth
- [ ] Configurer Dependabot sur le repo GitHub
- [ ] Choisir et configurer le gestionnaire de secrets
- [ ] Créer security@folia.app

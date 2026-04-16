# Conformité RGPD

## Contexte
Folia collecte des données personnelles (email, photos, localisation, comportement in-app). En tant qu'application européenne, la conformité RGPD est obligatoire.

## Données collectées

### Données fournies par l'utilisateur
- Email et mot de passe (compte)
- Nom d'affichage et photo de profil
- Photos de plantes uploadées
- Localisation (optionnelle — pour les échanges de boutures)
- Contenu publié (commentaires, collections)

### Données collectées automatiquement
- Logs d'utilisation (pages visitées, actions)
- Données techniques (OS, version app, IP)
- Notifications push (token appareil)

## Bases légales

| Donnée | Base légale |
|--------|-------------|
| Compte utilisateur | Exécution du contrat |
| Analytics | Intérêt légitime |
| Localisation | Consentement explicite |
| Notifications push | Consentement explicite |
| Cookies non essentiels | Consentement explicite |

## Droits des utilisateurs
- **Accès** : export de toutes ses données (format JSON)
- **Rectification** : modification depuis le profil
- **Effacement** : suppression du compte + purge des données sous 30 jours
- **Portabilité** : export structuré des collections et plantes
- **Opposition** : opt-out des analytics et communications marketing

## Mesures techniques
- Données chiffrées en transit (TLS 1.3) et au repos
- Mots de passe hashés (bcrypt, coût 12)
- Accès aux données restreint par rôle (RBAC)
- Logs d'audit pour les accès aux données sensibles
- Soft delete avec purge automatique après 30 jours

## Sous-traitants (DPA requis)
- Hébergeur cloud (données dans l'UE obligatoire)
- Service d'emails transactionnels
- Service de stockage des images
- Analytics (si tiers — préférer solution auto-hébergée)

## DPO
À désigner si le volume de traitement le requiert. Pour l'instant : Michel assume la responsabilité.

## Actions à faire
- [ ] Rédiger la politique de confidentialité complète
- [ ] Implémenter l'export des données utilisateur
- [ ] Mettre en place la bannière de consentement cookies
- [ ] Vérifier que l'hébergeur est bien dans l'UE
- [ ] Signer les DPA avec chaque sous-traitant

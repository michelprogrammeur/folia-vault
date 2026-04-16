# Politique cookies

## Statut
Brouillon — à faire valider avant mise en ligne.

## Qu'est-ce qu'un cookie ?
Un cookie est un petit fichier stocké sur l'appareil de l'utilisateur lors de la visite d'un site web. Il permet de mémoriser des préférences ou de suivre le comportement de navigation.

## Cookies utilisés par Folia

### Cookies essentiels (pas de consentement requis)
| Nom | Durée | Finalité |
|-----|-------|----------|
| `session_id` | Session | Authentification de l'utilisateur |
| `csrf_token` | Session | Protection contre les attaques CSRF |
| `cookie_consent` | 12 mois | Mémoriser le choix de consentement |

### Cookies analytiques (consentement requis)
| Nom | Durée | Finalité |
|-----|-------|----------|
| `_analytics` | 13 mois | Mesure d'audience anonymisée |

> Préférence : utiliser une solution analytics auto-hébergée (Plausible, Umami) pour éviter les cookies tiers et simplifier la conformité.

### Cookies marketing (consentement requis)
Aucun cookie marketing tiers prévu à ce stade. À réévaluer si des campagnes publicitaires sont lancées.

## Gestion du consentement
- Bannière de consentement affichée à la première visite
- Choix granulaire : essentiel / analytique / marketing
- Refus possible sans impact sur l'accès au service
- Modification du choix accessible depuis les paramètres du compte

## Application mobile
Les cookies ne s'appliquent pas aux apps mobiles. Les équivalents sont :
- **Identifiants de session** : stockés dans le secure storage de l'appareil
- **Analytics** : SDK analytics avec opt-out disponible
- **Notifications push** : consentement système (iOS/Android)

## Base légale
Directive ePrivacy + RGPD. Les cookies non essentiels nécessitent un consentement préalable, libre, éclairé et spécifique.

## À faire
- [ ] Choisir la solution analytics (Plausible recommandé — sans cookie)
- [ ] Implémenter la bannière de consentement (conformité CNIL)
- [ ] Faire auditer par un outil de conformité (Cookiebot scan, etc.)

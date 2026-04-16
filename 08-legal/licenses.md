# Licences open source

## Licence de Folia
À définir avant le lancement — options :
- **Propriétaire** : code source fermé, droits réservés
- **AGPL-3.0** : open source, toute modification doit être publiée (y compris SaaS)
- **MIT** : open source permissif

## Dépendances principales et leurs licences

### Backend
| Package | Licence | Contraintes |
|---------|---------|-------------|
| Fastify | MIT | Aucune |
| Drizzle ORM | Apache-2.0 | Aucune |
| Zod | MIT | Aucune |
| Vitest | MIT | Aucune |
| BullMQ | MIT | Aucune |
| Pino | MIT | Aucune |

### Frontend / Mobile
| Package | Licence | Contraintes |
|---------|---------|-------------|
| React Native | MIT | Aucune |
| Expo | MIT | Aucune |
| Vue.js | MIT | Aucune |

### Données botaniques
Les sources de données botaniques ont leurs propres licences :
- **GBIF** (Global Biodiversity Information Facility) : CC BY 4.0 — attribution requise
- **Plants of the World Online** : vérifier les conditions d'utilisation
- **Wikipedia / Wikidata** : CC BY-SA — attribution + partage dans les mêmes conditions

⚠️ L'utilisation de données botaniques tierces dans le catalogue Folia nécessite de vérifier les licences de chaque source et d'afficher les attributions requises.

## Obligations d'attribution
- Les licences MIT et Apache-2.0 n'imposent pas d'affichage public
- Les données CC BY nécessitent une mention de la source (ex: "Données botaniques : GBIF")
- Un écran "À propos / Licences" dans l'app est recommandé

## À faire
- [ ] Choisir la licence de Folia
- [ ] Générer un rapport de licences complet avec `license-checker`
- [ ] Vérifier les conditions d'utilisation de chaque source de données botaniques
- [ ] Ajouter l'écran "Licences" dans l'app

# ADR-002 — Clean Architecture par Feature (Vertical Slice)

## Statut
Accepté

## Contexte
Chaque service doit être organisé de façon à être maintenable sur le long terme. Il faut choisir une structure interne pour organiser le code.

## Options considérées

### Option 1 — Architecture en couches horizontales
Organisation par type technique : `controllers/`, `services/`, `repositories/`, `models/`.
- ✅ Familier, très répandu
- ❌ Une feature est dispersée dans tous les dossiers
- ❌ Les modifications d'une feature touchent N dossiers différents
- ❌ Couplage fort entre les couches

### Option 2 — Clean Architecture classique (Uncle Bob)
Couches concentriques : Entities → Use Cases → Interface Adapters → Frameworks.
- ✅ Séparation claire des responsabilités
- ✅ Domaine isolé des frameworks
- ❌ Beaucoup de boilerplate
- ❌ Difficile à appliquer strictement en pratique

### Option 3 — Clean Architecture par Feature (Vertical Slice) ✅
Organisation par feature, chaque feature contenant ses propres couches.
```
features/plant/
  domain/
  application/
  infrastructure/
  interface/
```
- ✅ Tout ce qui concerne une feature est au même endroit
- ✅ Facile d'ajouter ou supprimer une feature sans impacter les autres
- ✅ Les couches restent séparées à l'intérieur de chaque feature
- ✅ Scalable — chaque feature peut devenir un service indépendant
- ❌ Peut mener à de la duplication si mal appliqué

## Décision
**Clean Architecture par Feature (Vertical Slice).**

## Raisons
- Folia a des features bien délimitées (plant, collection, identification...)
- Une feature peut évoluer indépendamment des autres
- L'approche est cohérente avec les bounded contexts DDD
- Facilite l'extraction d'une feature en service séparé si nécessaire

## Conséquences
- Structure imposée dans chaque service : `features/<nom>/domain|application|infrastructure|interface`
- Le code partagé entre features va dans `shared/`
- Les features ne s'importent jamais directement entre elles
- Chaque feature suit les mêmes conventions de nommage

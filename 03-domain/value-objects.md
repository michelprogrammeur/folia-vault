# Value Objects — Règles de validation

> Un value object est immuable et défini par sa valeur, pas par son identité.
> Deux value objects avec les mêmes valeurs sont identiques.
> Chaque value object encapsule ses propres règles de validation.

---

## Identifiants

### `PlantId`
- Format : UUID v4
- Généré automatiquement à la création
- Immuable après création
- Exemple : `550e8400-e29b-41d4-a716-446655440000`

### `SpecimenId`
- Format : UUID v4
- Généré automatiquement à la création
- Immuable après création

### `UserId`
- Format : UUID v4
- Généré automatiquement à l'inscription
- Immuable après création

---

## Taxonomie

### `ScientificName`
- Format : binôme latin — Genre (majuscule) + épithète (minuscule)
- Minimum 3 caractères, maximum 100
- Doit contenir exactement un espace entre le genre et l'épithète
- Caractères autorisés : lettres latines, tirets, espace
- Exemples valides : `Monstera deliciosa`, `Ficus benjamina`
- Exemples invalides : `monstera deliciosa` (genre sans majuscule), `Monstera` (binôme incomplet)

### `TaxonomyReference`
- Contient : familyId, genusId, speciesId (tous obligatoires sauf au niveau famille)
- La hiérarchie doit être cohérente : le genre doit appartenir à la famille référencée
- Immuable — un changement de taxonomie crée un nouvel enregistrement

---

## Utilisateur

### `Email`
- Format RFC 5322 (validation standard)
- Converti en minuscules à la création
- Maximum 254 caractères
- Unique sur la plateforme
- Exemples valides : `user@example.com`, `user+tag@domain.co`

### `Pseudo`
- Minimum 3 caractères, maximum 30
- Caractères autorisés : lettres, chiffres, tirets, underscores
- Pas d'espaces, pas de caractères spéciaux
- Unique sur la plateforme
- Insensible à la casse pour la vérification d'unicité
- Exemples valides : `plantlover42`, `marie_jardine`

### `CityLocation`
- Contient : ville, département/région, pays, code postal
- **Jamais de coordonnées GPS précises**
- Résolution maximale : ville ou code postal
- Obligatoire pour accéder aux fonctionnalités géolocalisées

### `ProfileType`
- Valeurs autorisées : `BEGINNER`, `ENTHUSIAST`, `GARDENER`, `EXPERT`, `PROFESSIONAL`
- Modifiable par l'utilisateur
- Détermine l'interface affichée

---

## Entretien

### `Care`
- Arrosage (`WateringLevel`) : `LOW`, `MEDIUM`, `HIGH`
- Fréquence d'arrosage : entier positif, entre 1 et 365 jours
- Lumière (`LightLevel`) : `SHADE`, `INDIRECT`, `BRIGHT_INDIRECT`, `FULL_SUN`
- Humidité (`HumidityLevel`) : `LOW`, `MEDIUM`, `HIGH`
- Température min : entre -30°C et 50°C
- Température max : entre -30°C et 50°C, supérieure à la température min
- Sol : texte libre, maximum 200 caractères
- Fertilisation : texte libre, maximum 200 caractères

### `WateringFrequency`
- Entier positif en jours
- Minimum : 1 jour
- Maximum : 365 jours
- Peut varier selon la saison (été vs hiver)

---

## Plante

### `Appearance`
- Forme des feuilles (`LeafShape`) : `OVAL`, `LANCEOLATE`, `HEART`, `ROUND`, `LINEAR`, `LOBED`, `PINNATE`, `PALMATE`
- Couleur des feuilles (`LeafColor`) : `GREEN`, `DARK_GREEN`, `VARIEGATED`, `PURPLE`, `SILVER`, `GOLDEN`
- Texture des feuilles (`LeafTexture`) : `SMOOTH`, `WAXY`, `FUZZY`, `ROUGH`, `GLOSSY`
- Taille des feuilles (`LeafSize`) : `TINY`, `SMALL`, `MEDIUM`, `LARGE`, `EXTRA_LARGE`
- Couleur des fleurs (`FlowerColor`) : `WHITE`, `YELLOW`, `RED`, `PINK`, `PURPLE`, `ORANGE`, `BLUE`, `NONE`

### `Characteristics`
- Intérieur : booléen
- Extérieur : booléen (au moins un des deux doit être vrai)
- Toxique pour les humains : booléen
- Toxique pour les chiens : booléen
- Toxique pour les chats : booléen
- Comestible : booléen
- Mellifère : booléen
- Invasive : booléen
- Détails de toxicité : texte libre si toxique, maximum 500 caractères

### `GrowthRate`
- Valeurs autorisées : `SLOW`, `MEDIUM`, `FAST`

### `Difficulty`
- Valeurs autorisées : `EASY`, `MEDIUM`, `HARD`

### `PlantType`
- Valeurs autorisées : `PLANT`, `SHRUB`, `TREE`, `FLOWER`, `VEGETABLE`, `FRUIT`, `SUCCULENT`, `CACTUS`, `AQUATIC`, `EPIPHYTE`, `GRASS`, `FERN`, `PALM`

---

## Identification

### `ConfidenceScore`
- Nombre décimal entre 0.0 et 100.0
- Seuils :
  - ≥ 90% : identification fiable, affichage direct
  - 60–89% : identification probable, confirmation suggérée
  - < 60% : identification incertaine, validation humaine recommandée

### `IdentificationMethod`
- Valeurs autorisées : `PHOTO`, `AR_REALTIME`, `MANUAL`

---

## Gamification

### `Points`
- Entier positif ou zéro
- Ne peut jamais être négatif
- Plafond par action pour éviter les abus (défini par type d'action)

### `Streak`
- Entier positif ou zéro
- Repart à zéro si aucune action d'entretien dans les 24h
- Maximum théorique : illimité

### `Grade`
- Valeurs ordonnées : `SEED`, `SPROUT`, `SEEDLING`, `SHRUB`, `TREE`
- Uniquement croissant — on ne rétrograde jamais
- Chaque grade a un seuil de points minimum

---

## Géographie

### `Region`
- Contient : ville, code postal, département, pays
- Jamais de coordonnées GPS
- Utilisée pour l'affichage communautaire et les recommandations locales

### `ClimateZone`
- Basée sur la classification Köppen-Geiger
- Calculée automatiquement depuis la localisation de l'utilisateur
- Valeurs : `TROPICAL`, `DRY`, `TEMPERATE`, `CONTINENTAL`, `POLAR`
- Détermine le calendrier de jardinage et les alertes saisonnières

---

## Marketplace

### `ListingPrice`
- Montant décimal positif ou zéro (0 = don)
- Devise configurable (EUR par défaut)
- Maximum : 9999.99

### `ListingType`
- Valeurs autorisées : `SALE`, `TRADE`, `GIFT`

### `ListingStatus`
- Valeurs autorisées : `DRAFT`, `ACTIVE`, `RESERVED`, `CLOSED`
- Transitions autorisées : DRAFT→ACTIVE, ACTIVE→RESERVED, ACTIVE→CLOSED, RESERVED→ACTIVE, RESERVED→CLOSED

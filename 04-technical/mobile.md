# Mobile — Conventions et Stack

## Choix technologique

### React Native + Expo
Framework retenu pour l'application mobile Folia.

**Pourquoi React Native ?**
- Codebase partagée iOS + Android
- Écosystème JavaScript — cohérence avec le reste du projet (TypeScript partout)
- Large communauté, nombreuses librairies
- Performances suffisantes pour nos besoins

**Pourquoi Expo ?**
- Setup simplifié — pas de configuration native complexe au démarrage
- OTA updates (mises à jour sans passer par les stores)
- SDK riche : caméra, notifications, capteurs, AR...
- EAS Build pour les builds en CI/CD

**Alternatives écartées** :
- Flutter — Dart, écosystème différent, pas de cohérence avec le reste
- PWA — limitations natives trop importantes (AR, notifications, offline)
- Ionic — performances insuffisantes pour nos animations

---

## Structure du projet mobile

```
apps/mobile/
  src/
    features/
      plant/
        screens/             → PlantDetailScreen, PlantListScreen
        components/          → PlantCard, PlantCareCard
        hooks/               → usePlant, usePlantList
      collection/
        screens/
        components/
      identification/
        screens/             → CameraScreen, IdentificationResultScreen
      gamification/
        screens/
        components/
    navigation/              → React Navigation — stack, tabs, modals
    shared/
      api/                   → clients HTTP vers la gateway
      cache/                 → React Query / MMKV
      components/            → composants UI partagés
      hooks/                 → hooks utilitaires
      theme/                 → couleurs, typographie, spacing
  assets/
    images/
    fonts/
    animations/              → fichiers Lottie
```

---

## Navigation

**React Navigation v7**

```
Root Navigator
  ├── Auth Stack (non connecté)
  │     ├── OnboardingScreen
  │     ├── LoginScreen
  │     └── RegisterScreen
  └── App Tab Navigator (connecté)
        ├── Home Tab
        ├── Catalogue Tab
        ├── Camera Tab (identification AR)
        ├── Collection Tab
        └── Profile Tab
```

---

## Gestion des données

### React Query
- Fetching, caching et synchronisation des données serveur
- Cache local automatique avec invalidation
- Retry automatique en cas d'erreur réseau
- Background refetch quand l'app repasse en foreground

### MMKV
- Stockage local persistant haute performance
- Remplace AsyncStorage (beaucoup plus rapide)
- Utilisé pour : préférences, tokens, données offline

### Mode offline
- Les fiches plantes de la collection téléchargées localement
- Saisie d'événements carnet hors-ligne, sync à la reconnexion
- Indicateur visuel "hors-ligne" dans l'interface
- Gestion des conflits : last-write-wins pour les données de carnet

---

## Caméra et AR

### Identification par photo
- `expo-camera` pour la capture
- Upload vers le service `identification`
- Résultat affiché avec animation

### AR temps réel (V2)
- `expo-camera` + `ViroReact` ou `React Native Vision Camera`
- Analyse en streaming — frames envoyés toutes les 500ms
- Infos flottantes rendues en overlay sur le flux caméra
- Dégradation gracieuse si l'appareil ne supporte pas l'AR

---

## Notifications Push

- **Expo Notifications** pour la gestion unifiée iOS + Android
- Tokens push enregistrés dans le service `notification`
- Types de notifications :
  - Rappels d'entretien (locales si possible, push sinon)
  - Badges et grades (push)
  - Messages communauté (push groupés — digest)
  - Alertes météo (push)

---

## Performance et animations

- **React Native Reanimated 3** — animations 60fps sur le thread UI
- **Lottie** pour les animations complexes (onboarding, badges, succès)
- **FlashList** à la place de FlatList — performances supérieures pour les longues listes
- Images avec `expo-image` — cache natif, formats optimisés (WebP, AVIF)
- Éviter les re-renders inutiles : `memo`, `useCallback`, `useMemo` au bon endroit

---

## Déploiement

### Builds
- **EAS Build** pour les builds iOS et Android
- Builds automatiques via GitHub Actions à chaque merge sur `develop`

### Distribution
- **TestFlight** (iOS) et **Google Play Internal Testing** pour les bêtas
- **App Store** et **Google Play** pour la production

### OTA Updates (Expo)
- Mises à jour JavaScript sans validation des stores
- Utilisé pour les corrections urgentes et les petites améliorations
- Les changements natifs nécessitent toujours une soumission store

---

## Deep Linking et Universal Links

Les deep links permettent d'ouvrir l'app depuis un lien web ou une notification.

```
# Schéma custom (développement)
folia://plants/monstera-deliciosa
folia://collection/specimen-uuid
folia://challenges/spring-2025

# Universal Links (production — iOS) / App Links (Android)
https://folia.app/plants/monstera-deliciosa  → ouvre l'app si installée
```

### Configuration
- iOS : fichier `apple-app-site-association` sur le domaine
- Android : fichier `assetlinks.json` sur le domaine
- React Navigation gère le routing depuis les deep links

### Cas d'usage
- Notifications push → ouvrir directement le bon écran
- Partage d'une fiche plante → lien qui ouvre l'app
- Emails de rappel → bouton qui ouvre le carnet directement

## Conventions

- Même structure Clean Architecture par feature que les services back-end
- TypeScript strict partout
- Composants en PascalCase : `PlantCard.tsx`
- Hooks en camelCase préfixés par `use` : `usePlant.ts`
- Screens suffixés par `Screen` : `PlantDetailScreen.tsx`
- Styles avec `StyleSheet.create` ou NativeWind (Tailwind pour RN)

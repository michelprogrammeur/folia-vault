# Accessibilité

## Principe

L'accessibilité n'est pas une option — c'est une obligation légale (RGAA en France, WCAG 2.1 internationalement) et une opportunité d'atteindre plus d'utilisateurs.
Intégrer l'accessibilité dès le développement coûte 10x moins cher que de la rattraper après.

**Cible** : conformité **WCAG 2.1 niveau AA** sur web et mobile.

---

## Web (Astro / React)

### Contraste des couleurs
- Texte normal : ratio minimum **4.5:1**
- Texte large (18px+) : ratio minimum **3:1**
- Composants UI (boutons, inputs) : ratio minimum **3:1**
- Outil de vérification : [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)

### Structure sémantique HTML
```html
<!-- Correct — structure sémantique -->
<main>
  <h1>Monstera deliciosa</h1>
  <section aria-labelledby="care-title">
    <h2 id="care-title">Entretien</h2>
    <article>...</article>
  </section>
</main>

<!-- Incorrect — divs sans sémantique -->
<div class="main">
  <div class="title">Monstera deliciosa</div>
</div>
```

### Navigation clavier
- Tous les éléments interactifs accessibles au clavier (Tab, Enter, Escape, flèches)
- Focus visible sur tous les éléments interactifs
- Pas de piège au focus (modales accessibles)
- Skip link "Aller au contenu" en début de page

### Images
```html
<!-- Image informative -->
<img src="monstera.jpg" alt="Feuille de Monstera deliciosa avec ses fenestrations caractéristiques" />

<!-- Image décorative -->
<img src="decoration.jpg" alt="" role="presentation" />

<!-- Icône avec texte -->
<button>
  <Icon aria-hidden="true" />
  Arroser
</button>
```

### Formulaires
```html
<label for="scientific-name">Nom scientifique</label>
<input
  id="scientific-name"
  type="text"
  aria-describedby="scientific-name-hint"
  aria-required="true"
/>
<p id="scientific-name-hint">Format : Genre espèce (ex: Monstera deliciosa)</p>
```

### Notifications et alertes
```html
<!-- Annonce dynamique pour les lecteurs d'écran -->
<div role="status" aria-live="polite">
  Plante ajoutée à votre collection
</div>

<div role="alert" aria-live="assertive">
  Erreur : nom scientifique invalide
</div>
```

---

## Mobile (React Native)

### Propriétés d'accessibilité
```typescript
// Bouton accessible
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Arroser la Monstera deliciosa"
  accessibilityRole="button"
  accessibilityHint="Marque l'arrosage comme effectué aujourd'hui"
>
  <Icon name="water" />
</TouchableOpacity>

// Image accessible
<Image
  source={...}
  accessible={true}
  accessibilityLabel="Feuille de Monstera deliciosa"
/>

// Élément décoratif
<Image
  source={...}
  accessible={false}
  importantForAccessibility="no"
/>
```

### VoiceOver (iOS) et TalkBack (Android)
- Tester avec VoiceOver sur iOS (Réglages → Accessibilité → VoiceOver)
- Tester avec TalkBack sur Android (Paramètres → Accessibilité → TalkBack)
- Ordre de lecture logique et cohérent avec l'ordre visuel

### Taille de texte
- Respecter la taille de texte système de l'utilisateur
- Pas de taille fixe en pixels — utiliser les unités relatives
- Interface utilisable jusqu'à 200% de zoom

---

## Modes d'accessibilité

### Mode daltonien
Folia propose 3 variantes pour les types de daltonisme courants :
- Deutéranopie (vert-rouge)
- Protanopie (rouge)
- Tritanopie (bleu-jaune)

Ne jamais utiliser la couleur seule pour transmettre une information :
```
// Mauvais — couleur seule
<StatusDot color="red" />

// Correct — couleur + texte/icône
<StatusDot color="red" label="Danger" icon="warning" />
```

### Réduction des animations
```typescript
// Respecter la préférence système "Réduire les animations"
import { useReducedMotion } from 'react-native-reanimated'

function AnimatedCard() {
  const reduceMotion = useReducedMotion()

  const animatedStyle = useAnimatedStyle(() => ({
    transform: reduceMotion
      ? []
      : [{ scale: withSpring(scale.value) }]
  }))
}
```

### Mode haut contraste
- Détecter `AccessibilityInfo.isHighContrastEnabled()` sur mobile
- Adapter les couleurs en conséquence

---

## Tests d'accessibilité

### Automatisés
- **axe-core** (web) — intégré dans les tests Vitest/Jest
- **eslint-plugin-jsx-a11y** — règles ESLint pour l'accessibilité JSX
- **Lighthouse** — audit accessibilité dans le pipeline CI

### Manuels
- Navigation clavier complète sur les parcours critiques
- Test avec VoiceOver (iOS) et TalkBack (Android)
- Test avec zoom 200%
- Test en mode daltonien

### Checklist avant chaque release
- [ ] Contraste vérifié sur les nouvelles couleurs
- [ ] Nouvelles images ont un texte alternatif
- [ ] Nouveaux formulaires ont des labels associés
- [ ] Navigation clavier testée sur les nouveaux écrans
- [ ] Pas de contenu uniquement accessible à la souris

---

## Ressources

- [WCAG 2.1 — W3C](https://www.w3.org/TR/WCAG21/)
- [RGAA 4.1 — France](https://accessibilite.numerique.gouv.fr/)
- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [axe-core](https://github.com/dequelabs/axe-core)

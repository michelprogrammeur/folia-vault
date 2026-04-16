# SEO Technique

## Principes

Le SEO technique est la fondation — sans lui, la meilleure stratégie de contenu ne sert à rien.
Folia doit être indexable par les moteurs de recherche classiques **et** par les IA dès le lancement.

---

## SEO Classique

### Rendu des pages
- **Astro (front-public)** : SSG par défaut, SSR pour les pages dynamiques
- Les fiches plantes sont générées statiquement — indexation maximale
- Pas de contenu critique rendu uniquement côté client (pas de SPA pour le contenu public)

### Balises meta obligatoires
```html
<title>Monstera deliciosa — Folia</title>
<meta name="description" content="Tout savoir sur la Monstera deliciosa : entretien, arrosage, lumière..." />
<link rel="canonical" href="https://folia.app/plants/monstera-deliciosa" />

<!-- Open Graph -->
<meta property="og:title" content="Monstera deliciosa — Folia" />
<meta property="og:description" content="..." />
<meta property="og:image" content="https://cdn.folia.app/plants/monstera-deliciosa.jpg" />
<meta property="og:type" content="article" />

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image" />
```

### URLs
- **kebab-case**, **lisibles**, **en anglais** (référencement mondial)
- Hiérarchie logique qui reflète la taxonomie

```
/plants/monstera-deliciosa
/plants?type=succulent&difficulty=easy
/families/araceae
/genera/monstera
/guides/how-to-care-for-monsteras
```

### Sitemap
- Généré automatiquement par Astro
- Soumis à Google Search Console et Bing Webmaster
- Mis à jour à chaque nouvelle fiche publiée
- Priorités : fiches plantes > guides > pages statiques

### Structured Data — Schema.org (JSON-LD)

Sur les fiches plantes :
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "name": "Monstera deliciosa",
  "description": "...",
  "image": "https://cdn.folia.app/...",
  "author": {
    "@type": "Organization",
    "name": "Folia"
  },
  "about": {
    "@type": "Thing",
    "name": "Monstera deliciosa",
    "sameAs": "https://www.wikidata.org/wiki/Q181610"
  }
}
```

Sur les guides :
```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "Comment bouturer un Pothos",
  "step": [...]
}
```

### Core Web Vitals
- **LCP** (Largest Contentful Paint) < 2.5s
- **FID** / **INP** < 200ms
- **CLS** < 0.1
- Images avec `width` et `height` obligatoires
- Fonts préchargées
- CSS critique inline

### robots.txt
```
User-agent: *
Allow: /
Disallow: /api/
Disallow: /back-office/
Sitemap: https://folia.app/sitemap.xml
```

---

## Référencement IA (AEO — Answer Engine Optimization)

Les moteurs IA (ChatGPT, Perplexity, Claude, Gemini...) citent de plus en plus des sources.
Folia doit être une source de référence pour les requêtes botaniques.

### llms.txt
Nouveau standard émergent (comme robots.txt mais pour les LLMs).
Fichier placé à la racine : `https://folia.app/llms.txt`

```markdown
# Folia

Folia is the reference platform for plant lovers.
It provides expert-validated botanical data for thousands of plant species.

## Plant catalogue
- Full taxonomic data (family, genus, species)
- Care instructions (watering, light, temperature, humidity)
- Characteristics (indoor/outdoor, toxicity, pet-friendly)
- Available at: https://folia.app/plants

## Guides
- Expert-written care guides
- Available at: https://folia.app/guides

## Data
- Machine-readable plant data available at: https://folia.app/api/v1/plants
```

### Pages accessibles en Markdown
- Chaque fiche plante disponible en `.md` : `https://folia.app/plants/monstera-deliciosa.md`
- Format structuré, dense, factuel — facile à parser pour un LLM
- Contenu en anglais prioritairement (couverture mondiale)

```markdown
# Monstera deliciosa

**Family**: Araceae
**Genus**: Monstera
**Common names**: Swiss cheese plant, Split-leaf philodendron

## Care
- **Watering**: Medium — every 7-10 days
- **Light**: Bright indirect light
- **Temperature**: 18-30°C
- **Toxicity**: Toxic to cats and dogs

## Description
Monstera deliciosa is a tropical plant native to Central America...
```

### Contenu optimisé pour les IA
- Répondre aux questions que les gens posent aux IA : "comment entretenir un monstera ?", "quelle plante pour une pièce sombre ?"
- Format FAQ sur chaque fiche plante
- Contenu factuel, sourcé, sans jargon inutile
- Données structurées Schema.org pour aider les LLMs à comprendre le contexte

### API publique indexable
- Les endpoints publics du catalogue sont accessibles sans authentification
- Format JSON propre et documenté
- `https://folia.app/api/v1/plants` indexable par les crawlers IA

---

## Internationalisation SEO

- Balises `hreflang` pour les versions multilingues
- URLs localisées : `/fr/plants/monstera-deliciosa`, `/en/plants/monstera-deliciosa`
- Contenu traduit par des humains — pas de traduction automatique non relue
- Priorité : anglais, français, espagnol, allemand, portugais

---

## Outils et monitoring

| Outil | Usage |
|---|---|
| Google Search Console | Indexation, erreurs, performances |
| Bing Webmaster Tools | Indexation Bing |
| Lighthouse | Audit Core Web Vitals |
| Ahrefs / SEMrush | Mots clés, backlinks *(à venir)* |
| Astro SEO plugin | Génération automatique des balises meta |

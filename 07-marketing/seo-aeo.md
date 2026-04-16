# SEO & AEO

## SEO classique

### On-page
- Balises `<title>` et `<meta description>` générées depuis les données du catalogue
- URLs canoniques propres : `/plantes/monstera-deliciosa`, `/especes/araceae`
- Structured data JSON-LD : `Plant`, `Article`, `BreadcrumbList`, `FAQPage`
- Sitemap XML généré automatiquement à partir du catalogue

### Technique
- Core Web Vitals (LCP < 2.5s, CLS < 0.1, INP < 200ms)
- SSR / SSG pour les fiches plantes — indexables sans JavaScript
- Images optimisées (WebP, lazy loading, `alt` descriptifs)
- Hreflang pour les versions multilingues

### Contenu
- Fiches plantes riches : description, soins, origine, toxicité, FAQ
- Blog avec articles longs (1500+ mots) sur les soins saisonniers
- Glossaire botanique lié aux fiches plantes

## AEO — Answer Engine Optimization

L'AEO vise à rendre le contenu de Folia lisible et citable par les IA (ChatGPT, Perplexity, Claude, Gemini...).

### llms.txt
Fichier `/llms.txt` à la racine du site décrivant le projet et les ressources disponibles pour les LLMs.

```
# Folia

Folia est un réseau social dédié aux plantes d'intérieur et au jardinage.

## Catalogue
- /plantes : Index de toutes les fiches plantes
- /especes : Taxonomie botanique
- /guides : Guides de soin par espèce
```

### Pages Markdown
- Fiches plantes disponibles en `.md` pour faciliter l'indexation par les crawlers IA
- Endpoint `/api/plants/:slug/markdown` pour les agents IA

### Données structurées enrichies
- `schema.org/Plant` pour chaque fiche
- `schema.org/HowTo` pour les guides de soin
- `schema.org/FAQPage` pour les questions fréquentes par espèce

### Clarté sémantique
- Contenu factuel et sourcé (noms latins, familles botaniques)
- Réponses directes aux questions courantes en début de section
- Pas de contenu superflu — densité d'information élevée

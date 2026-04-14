# Fonctionnalité — Identification de plantes

## Description
Identifier une plante instantanément via la caméra du smartphone.

## Modes

### V1 — Photo statique
- L'utilisateur prend une photo ou en importe une depuis sa galerie
- Résultat : nom scientifique, nom commun, lien vers la fiche plante
- API externe : PlantNet API ou Google Vision

### V2 — AR temps réel
- L'utilisateur pointe sa caméra vers une plante
- Les informations s'affichent en temps réel en superposition (AR)
- Infos flottantes : nom, niveau de difficulté, toxicité, lien vers la fiche

## Résultat d'identification
- Nom scientifique + nom(s) commun(s)
- Niveau de confiance (%)
- Lien direct vers la fiche plante dans le catalogue
- Bouton "Ajouter à ma collection"
- Historique des identifications

## Limites MVP
- Gratuit : N identifications par jour (ex: 5)
- Premium : illimité

## MVP
- Identification par photo avec API externe
- Historique des identifications
- Lien vers la fiche plante

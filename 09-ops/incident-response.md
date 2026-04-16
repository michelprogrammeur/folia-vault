# Gestion des incidents

## Niveaux de sévérité

| Niveau | Définition | Temps de réponse |
|--------|-----------|-----------------|
| P1 — Critique | Service totalement indisponible | < 15 minutes |
| P2 — Majeur | Fonctionnalité clé dégradée | < 1 heure |
| P3 — Mineur | Bug impactant quelques utilisateurs | < 24 heures |
| P4 — Faible | Problème cosmétique ou marginal | Prochaine release |

## Processus de réponse

### 1. Détection
- Alerte automatique (Grafana, health check)
- Signalement utilisateur (support, réseaux sociaux)
- Découverte interne

### 2. Qualification
- Identifier le service impacté
- Estimer le nombre d'utilisateurs affectés
- Attribuer un niveau de sévérité (P1 à P4)

### 3. Mitigation immédiate
- **Rollback** : revenir à la version précédente si le déploiement est en cause
- **Feature flag** : désactiver la fonctionnalité problématique
- **Scaling** : augmenter les ressources si c'est un problème de charge
- **Maintenance** : afficher une page de maintenance si nécessaire

### 4. Communication
- P1/P2 : mettre à jour la status page dans les 15 minutes
- Informer les utilisateurs impactés si données affectées
- Communication interne sur Slack / Discord

### 5. Résolution
- Déployer le correctif
- Vérifier la stabilité (métriques revenues à la normale)
- Lever l'alerte et mettre à jour la status page

### 6. Post-mortem
Pour tout incident P1/P2 : rédiger un post-mortem dans les 48h.

## Template post-mortem
```
## Incident — [titre]
**Date** : 
**Durée** : 
**Impact** : X utilisateurs affectés

### Chronologie
- HH:MM — Détection
- HH:MM — Mitigation
- HH:MM — Résolution

### Cause racine

### Ce qui a bien fonctionné

### Ce qui a mal fonctionné

### Actions correctives
- [ ] Action 1 — Responsable — Deadline
```

## Outils
- **Status page** : Statuspage.io ou Instatus
- **Alertes** : Grafana Alerting
- **Communication** : Slack / Discord
- **Runbooks** : liens depuis les alertes Grafana vers les procédures de résolution

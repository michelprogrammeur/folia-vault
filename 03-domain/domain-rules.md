# Règles Métier — Domain Rules

> Règles transversales qui gouvernent le comportement de Folia.
> Ces règles sont indépendantes de l'implémentation technique.
> En cas de doute sur un comportement, ce fichier fait référence.

---

## Sécurité et vie privée

**R-SEC-01** — La localisation d'un utilisateur ne doit jamais être plus précise qu'une ville ou un code postal. Aucune coordonnée GPS ne doit être stockée ni partagée publiquement.

**R-SEC-02** — Un utilisateur supprimé (demande RGPD) voit toutes ses données personnelles effacées dans les 30 jours. Ses contributions publiques (observations, articles) sont conservées mais anonymisées sous le pseudo "Utilisateur supprimé".

**R-SEC-03** — Les données de santé des plantes (carnet, diagnostics) sont privées par défaut et ne peuvent être partagées que sur action explicite de l'utilisateur.

**R-SEC-04** — Un mineur (< 18 ans) ne peut pas accéder aux fonctionnalités de marketplace ni de messagerie privée sans consentement parental.

---

## Catalogue et taxonomie

**R-CAT-01** — Une plante ne peut être publiée que si elle possède : un nom scientifique valide, une espèce référencée, des données d'entretien complètes (arrosage, lumière, température min/max) et au moins une photo.

**R-CAT-02** — Le nom scientifique d'une plante doit être unique dans le catalogue. Les synonymes sont stockés séparément et redirigent vers la fiche principale.

**R-CAT-03** — Une fiche espèce ne peut pas être supprimée si des spécimens actifs y font référence dans des collections utilisateurs.

**R-CAT-04** — Toute plante marquée comme toxique (humains, chiens ou chats) doit afficher un avertissement visible en priorité, avant toute autre information.

**R-CAT-05** — Le contenu du catalogue est validé par au moins un expert avant publication. Le contenu non validé est visible uniquement en mode "brouillon" pour les administrateurs.

---

## Collection et carnet

**R-COL-01** — Un utilisateur gratuit ne peut pas avoir plus de 10 spécimens actifs dans sa collection. Un utilisateur premium n'a pas de limite.

**R-COL-02** — La date de plantation d'un spécimen ne peut pas être postérieure à la date du jour.

**R-COL-03** — Les entrées du carnet sont immuables une fois créées. On peut ajouter une nouvelle entrée pour corriger, jamais modifier l'existante.

**R-COL-04** — Un spécimen supprimé passe en soft delete — son historique (journal, photos) est conservé pendant 90 jours avant suppression définitive.

**R-COL-05** — Les rappels d'entretien sont automatiquement ajustés selon la saison et la zone climatique de l'utilisateur.

---

## Identification

**R-IDN-01** — Un utilisateur gratuit est limité à 5 identifications par jour. Un utilisateur premium n'a pas de limite.

**R-IDN-02** — Une identification avec un score de confiance inférieur à 60% doit proposer une validation par un expert ou la communauté.

**R-IDN-03** — Une identification est immuable une fois créée. Si l'utilisateur pense que le résultat est incorrect, il soumet une nouvelle identification ou signale l'erreur.

**R-IDN-04** — Les images soumises pour identification sont conservées maximum 30 jours puis supprimées, sauf si l'utilisateur crée une observation depuis l'identification.

---

## Gamification

**R-GAM-01** — La progression de grade est uniquement croissante. Un utilisateur ne peut pas perdre son grade, quelle que soit son inactivité.

**R-GAM-02** — Les points attribués pour une même action sont plafonnés par période (ex: max 50 points/jour pour les arrosages) pour éviter les abus.

**R-GAM-03** — Un badge obtenu ne peut pas être retiré, sauf en cas de violation des conditions d'utilisation.

**R-GAM-04** — Un défi expiré ne peut plus être complété, même si l'utilisateur avait atteint les objectifs avant l'expiration.

**R-GAM-05** — Le streak se remet à zéro si aucune action d'entretien n'est enregistrée pendant 24h consécutives. Un seul "joker" de protection par mois est disponible pour les utilisateurs premium.

---

## Communauté et social

**R-SOC-01** — Un utilisateur peut signaler un contenu inapproprié. Après 3 signalements distincts, le contenu est automatiquement masqué en attente de modération.

**R-SOC-02** — Le statut "Expert vérifié" est attribué manuellement par l'équipe Folia après vérification des compétences. Il ne peut pas être auto-attribué.

**R-SOC-03** — Une réponse marquée comme "validée par un expert" ne peut l'être que par un utilisateur avec le statut Expert vérifié.

**R-SOC-04** — La messagerie privée est désactivée par défaut. L'utilisateur doit l'activer explicitement dans ses paramètres.

**R-SOC-05** — Un utilisateur bloqué ne peut plus voir le profil, la collection ni les publications du bloqueur.

---

## Marketplace

**R-MKT-01** — Une transaction ne peut être initiée que si l'annonce est en statut ACTIVE.

**R-MKT-02** — Un utilisateur ne peut pas transacter avec lui-même.

**R-MKT-03** — Une annonce sans activité depuis 90 jours est automatiquement clôturée avec notification au vendeur.

**R-MKT-04** — La notation d'une transaction n'est possible que dans les 30 jours suivant la clôture. Passé ce délai, la fenêtre de notation est fermée.

**R-MKT-05** — Un vendeur certifié doit maintenir une note moyenne ≥ 3.5/5 pour conserver son statut certifié.

---

## Observations et biodiversité

**R-GEO-01** — Une observation ne peut être soumise qu'avec une identification associée (score ≥ 60% ou validation manuelle).

**R-GEO-02** — Une observation validée est immuable. Une erreur donne lieu à une nouvelle observation corrigée, pas à une modification.

**R-GEO-03** — Les observations contribuées à des bases de données scientifiques partenaires sont anonymisées avant export.

---

## Contenu éditorial

**R-CNT-01** — Tout contenu soumis par un utilisateur (photo, article, traduction) doit être validé avant publication. Le délai de modération cible est de 48h.

**R-CNT-02** — Un article refusé ne disparaît pas — il est retourné à l'auteur avec les raisons du refus pour correction.

**R-CNT-03** — Le crédit de l'auteur (photo, article, traduction) est permanent et ne peut pas être supprimé même si l'utilisateur quitte la plateforme (il devient "Contributeur anonyme").

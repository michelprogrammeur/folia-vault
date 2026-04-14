# Glossaire — Ubiquitous Language

> Termes métier utilisés dans toute la plateforme Folia — du code au business.
> Chaque terme a une définition précise et unique. En cas de doute, ce fichier fait référence.

---

## Plantes et taxonomie

**Famille** (`Family`)
Niveau taxonomique regroupant plusieurs genres. Ex: Araceae (famille des arums).

**Genre** (`Genus`)
Subdivision d'une famille regroupant des espèces proches. Ex: Monstera.

**Espèce** (`Species`)
Unité de base de la classification. Définie par un binôme latin (genre + épithète). Ex: *Monstera deliciosa*.

**Cultivar**
Variété cultivée sélectionnée par l'homme. Ex: *Monstera deliciosa* 'Thai Constellation'.

**Hybride**
Croisement entre deux espèces ou genres différents. Noté avec un × dans le nom scientifique.

**Synonyme**
Ancien nom scientifique d'une espèce, remplacé par la nomenclature actuelle mais encore en usage.

**Nom vernaculaire**
Nom commun d'une plante dans une langue donnée. Une espèce peut avoir plusieurs noms vernaculaires.

---

## Collection et carnet

**Collection**
Ensemble des plantes qu'un utilisateur possède et suit dans Folia.

**Spécimen** (`Specimen`)
Un exemplaire physique d'une plante dans la collection d'un utilisateur. Distinct de l'espèce — un utilisateur peut avoir plusieurs spécimens de la même espèce.

**Carnet** (`Journal`)
Historique complet d'un spécimen : événements, soins, photos, maladies, traitements.

**Événement** (`JournalEntry`)
Action enregistrée dans le carnet d'un spécimen : arrosage, rempotage, taille, maladie, traitement, floraison...

**Rappel** (`Reminder`)
Notification planifiée liée à l'entretien d'un spécimen.

**Plantation** (`Planting`)
Date à laquelle un spécimen a été planté ou acquis — point de départ du calcul de l'âge.

---

## Identification

**Identification** (`Identification`)
Résultat d'une tentative de reconnaissance d'une espèce via photo ou AR. Contient l'espèce proposée, le niveau de confiance et la méthode utilisée.

**Observation** (`Observation`)
Identification validée et géolocalisée, contribuée à la base de données de biodiversité communautaire.

**Niveau de confiance** (`ConfidenceScore`)
Pourcentage indiquant la certitude de l'identification. En dessous d'un seuil, une validation humaine est suggérée.

---

## Communauté

**Profil** (`Profile`)
Représentation publique d'un utilisateur : grade, collection publique, badges, contributions.

**Grade** (`Grade`)
Niveau de progression d'un utilisateur dans Folia, basé sur ses actions et contributions.

**Badge** (`Badge`)
Récompense visuelle obtenue en accomplissant une action spécifique ou un défi.

**Expert** (`Expert`)
Utilisateur vérifié avec des compétences botaniques reconnues. Peut valider des contenus et répondre en priorité aux questions.

**Contribution** (`Contribution`)
Tout contenu soumis par un utilisateur pour enrichir la plateforme : photo, observation, article, traduction.

**Échange** (`Trade`)
Transaction entre deux utilisateurs pour échanger ou donner des plantes, boutures ou graines.

---

## Gamification

**Points** (`Points`)
Monnaie virtuelle gagnée par les actions de l'utilisateur. Contribue à la progression de grade.

**Défi** (`Challenge`)
Objectif à accomplir dans un délai donné, individuel ou collectif, avec récompense à la clé.

**Saison** (`Season`)
Période de jeu thématique calée sur le calendrier naturel, avec défis et récompenses exclusives.

**Streak**
Série de jours consécutifs avec une action d'entretien effectuée.

---

## Contenu

**Fiche plante** (`PlantCard`)
Page de référence d'une espèce dans le catalogue : taxonomie, entretien, caractéristiques, médias.

**Guide** (`Guide`)
Article pratique rédigé par un expert ou la communauté sur un sujet précis.

**Événement communautaire** (`CommunityEvent`)
Sortie, conférence, marché ou atelier organisé par Folia ou la communauté, visible sur la carte.

---

## Propagation

**Bouture** (`Cutting`)
Fragment prélevé sur une plante mère (tige, feuille, racine) pour produire un nouvel individu identique.

**Marcottage** (`Layering`)
Technique de propagation où une tige est enracinée avant d'être séparée de la plante mère. Taux de réussite élevé.

**Semis** (`Seeding`)
Multiplication par graines. Peut produire des individus légèrement différents de la plante mère (variabilité génétique).

**Greffage** (`Grafting`)
Union de deux parties de plantes différentes : le porte-greffe (racines) et le greffon (partie aérienne). Utilisé pour les fruitiers et rosiers notamment.

**Division** (`Division`)
Séparation d'une plante en plusieurs parties possédant chacune racines et feuilles. Technique pour les touffes et rhizomes.

**Taux de réussite** (`SuccessRate`)
Pourcentage moyen de réussite d'une technique de propagation pour une espèce donnée, calculé sur les retours communautaires.

---

## Santé et diagnostic

**Symptôme** (`Symptom`)
Signe visible d'un problème sur une plante : feuilles jaunes, taches, flétrissement, moisissures...

**Pathologie** (`Disease`)
Maladie identifiée affectant une plante : fongique, bactérienne, virale ou liée à des ravageurs.

**Carence** (`Deficiency`)
Manque d'un nutriment essentiel visible sur la plante. Ex: carence en fer (jaunissement des jeunes feuilles), carence en azote (jaunissement général).

**Traitement** (`Treatment`)
Solution appliquée pour résoudre un problème sanitaire : produit naturel, chimique, taille, isolement...

**Ravageur** (`Pest`)
Organisme nuisible s'attaquant à la plante : cochenilles, araignées rouges, pucerons, aleurodes...

**Diagnostic** (`Diagnostic`)
Analyse d'une pathologie ou carence à partir de photos ou symptômes décrits, effectuée par l'IA ou un expert.

---

## Marketplace

**Annonce** (`Listing`)
Offre publiée par un utilisateur pour vendre, échanger ou donner une plante, bouture ou graine.

**Transaction** (`Transaction`)
Échange finalisé entre deux utilisateurs suite à une annonce.

**Notation** (`Rating`)
Évaluation laissée après une transaction sur la fiabilité et la qualité du vendeur ou donneur.

**Vendeur certifié** (`CertifiedSeller`)
Professionnel (pépiniériste, fleuriste, paysagiste) dont l'identité et la qualité ont été vérifiées par Folia.

---

## Gamification — compléments

**Carte collectionnable** (`CollectableCard`)
Illustration botanique unique débloquée lors de l'identification ou l'ajout d'une espèce à sa collection.

**Récompense** (`Reward`)
Avantage concret débloqué par les points ou grades : promotion partenaire, fonctionnalité premium temporaire, badge exclusif.

**Succès caché** (`HiddenAchievement`)
Badge secret découvert par accident en accomplissant une combinaison d'actions non documentée.

---

## Géographie

**Zone climatique** (`ClimateZone`)
Classification climatique de la localisation d'un utilisateur (système Köppen). Détermine les recommandations saisonnières.

**Région** (`Region`)
Découpage géographique utilisé pour la cartographie communautaire — ville ou département, jamais GPS précis.

**Lieu botanique** (`BotanicPlace`)
Parc, jardin botanique ou réserve naturelle référencé dans Folia. Peut être visité pour gagner des points.

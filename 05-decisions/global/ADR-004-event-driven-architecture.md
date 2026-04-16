# ADR-004 — Architecture Event-Driven entre les services

## Statut
Accepté (implémentation progressive)

## Contexte
Les services de Folia doivent communiquer entre eux. Il faut choisir le mode de communication inter-services : synchrone (REST) ou asynchrone (événements).

## Options considérées

### Option 1 — Communication synchrone uniquement (REST)
Chaque service appelle directement les autres via HTTP.
- ✅ Simple à implémenter et déboguer
- ✅ Résultat immédiat
- ❌ Couplage fort — si un service tombe, tous ses appelants sont bloqués
- ❌ Latence en cascade — une requête attend toutes les réponses
- ❌ Difficile à scaler indépendamment

### Option 2 — Communication asynchrone uniquement (événements)
Tous les services communiquent via un message broker.
- ✅ Découplage total
- ✅ Résilience — un service peut être indisponible temporairement
- ❌ Complexité accrue
- ❌ Cohérence éventuelle — pas de résultat immédiat
- ❌ Difficile à déboguer

### Option 3 — Hybride : synchrone pour les critiques, asynchrone pour le reste ✅
- REST synchrone quand une réponse immédiate est nécessaire
- Événements asynchrones pour les effets de bord et la propagation

- ✅ Pragmatique — chaque communication utilise le bon outil
- ✅ Démarrer simple (BullMQ), évoluer vers Kafka si nécessaire
- ✅ Les services restent découplés pour les opérations non critiques

## Décision
**Architecture hybride avec progression BullMQ → RabbitMQ → Kafka.**

### Phase MVP — BullMQ + Redis
Suffisant pour démarrer, Redis déjà dans la stack.

### Phase V2 — RabbitMQ
Quand plusieurs services consomment les mêmes événements (fan-out).

### Phase V3 — Kafka
Quand le volume et le besoin de replay le justifient.

## Raisons
- Commencer simple et évoluer selon les besoins réels (YAGNI)
- Redis est déjà nécessaire pour le cache — BullMQ ne coûte rien de plus
- L'architecture hybride est un standard de l'industrie des microservices

## Conséquences
- Les services émettent des événements domaine après chaque mutation
- Les événements suivent un format standard documenté dans `03-domain/domain-events.md`
- Les consommateurs sont idempotents — un événement peut arriver plusieurs fois
- Les erreurs de traitement sont envoyées en Dead Letter Queue
- La communication synchrone reste pour : vérification de token, matching d'identification

# Event pipeline sur Cloud Run et Pub/Sub

**Question.** Comment absorber un gros volume d'événements (cible : 10 000 messages/s pendant 5 minutes) avec
Cloud Run et Pub/Sub, et que valent push et pull l'un contre l'autre en débit, latence, résilience et coût ?

**Statut.** En cours. Squelette posé, rien de déployé.

## Architecture

```
k6 / Cloud Run Job (générateur)
        │  HTTP
        ▼
ingest (Cloud Run service) ──publish, batché──▶ topic `events`
                                                    │
               ┌────────────────────────────────────┼──────────────────────────┐
               ▼                                    ▼                          ▼
   sub push `events-push`               sub pull `events-pull`      sub BigQuery `events-bq`
   → processor-push (Cloud Run service) → processor-pull (Cloud Run   (sink sans code : comptage,
     scalé par les requêtes                worker pool, streaming pull) doublons, latence)
               │ échec x5                           │ échec x5
               └──────────────▶ topic `events-dlq` ◀┘
                                       │
                               sub `events-dlq` → alerte + Cloud Run Job de rejeu
```

- `ingest` : seul composant exposé. Valide, publie en batch, propage un identifiant de trace.
- `processor-push` : Pub/Sub appelle Cloud Run en HTTP avec un token OIDC. Scaling par requêtes, concurrence 80.
- `processor-pull` : worker pool Cloud Run, CPU toujours alloué, streaming pull avec contrôle de flux. Scaling piloté sur `num_undelivered_messages`.
- Sink BigQuery natif : vérité pour compter ce qui est passé et mesurer la latence de bout en bout.
- DLQ après 5 tentatives, Cloud Run Job de rejeu.
- Idempotence par `event_id` : écriture `create` dans Firestore, l'échec signifie doublon.
- Infra Terraform, déploiement GitHub Actions via Workload Identity Federation, images sur Artifact Registry.
- Garde-fous : `max-instances` partout, alerte budget, `make down` détruit tout.

## Expériences

| # | Expérience | Mesure | Résultat |
|---|---|---|---|
| 1 | Push vs pull à 10 000 msg/s pendant 5 min | débit, p50/p99, instances, coût | à faire |
| 2 | Backpressure : processors coupés 2 min puis relancés | temps de résorption, âge du plus vieux message | à faire |
| 3 | Ordering keys | perte de débit | à faire |
| 4 | Exactly-once sur l'abonnement pull | débit, doublons dans BigQuery | à faire |
| 5 | Cold start : min-instances 0 vs 1, startup CPU boost | p99 au démarrage | à faire |
| 6 | 5 % d'erreurs injectées | messages en DLQ, backoff, zéro perte | à faire |

## Arborescence

```
infra/                    Terraform : topics, abonnements, Cloud Run, IAM, Artifact Registry, budget
services/ingest           Node.js
services/processor-push   Node.js
services/processor-pull   Node.js
services/replay-job       Node.js
load/                     scénarios k6
docs/                     résultats et captures
```

## Ce que j'en retiens

À compléter une fois les mesures faites.

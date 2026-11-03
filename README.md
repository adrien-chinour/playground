# Playground

Expériences techniques, une par dossier. Chaque dossier répond à une question précise et son README donne la
réponse, chiffrée quand c'est possible. Le code est là pour être relu, pas pour être réutilisé tel quel.

| Thème | Expérience | Question | Réponse courte | Date |
|---|---|---|---|---|
| GCP | [event-pipeline](gcp/event-pipeline/) | Comment absorber un gros volume d'événements avec Cloud Run et Pub/Sub, en push et en pull ? | En cours | 2026-11 |
| GCP | [php-runtimes](gcp/php-runtimes/) | Quel runtime PHP tient le mieux la charge sur Cloud Run et Cloud Functions ? | FrankenPHP en conteneur : 1 646 req/s, contre 78 req/s pour Apache mod_php sans concurrence et 1 374 req/s pour une Cloud Function PHP | 2024-02 |
| GCP | [messaging](gcp/messaging/) | Cloud Tasks ou Pub/Sub pour traiter des commandes e-commerce de façon asynchrone ? | Les deux lissent la charge ; Cloud Tasks met 15 min à écouler 2 min de pic avec 10 instances max, et les handlers doivent être idempotents dans les deux cas | 2024-01 |
| Web | [varnish](web/varnish/) | Comment mettre en cache un site Symfony avec Varnish, ESI compris, et l'observer ? | VCL avec `Surrogate-Capability` et `do_esi`, stack Prometheus + Loki + Grafana pour `varnishstat` et `varnishncsa`, charge k6 | 2024-11 |

## Conventions

- Un dossier par expérience, rangé par thème : `gcp/`, `web/`, d'autres viendront.
- Le README de chaque dossier suit le même plan : question, ce qui a été fait, résultat, ce que j'en retiens.
- Les ressources cloud créées pour une expérience sont détruites à la fin. Les identifiants de projet qui restent dans le code pointent vers des projets supprimés.
- Les expériences importées depuis d'anciens dépôts gardent leur historique de commits.

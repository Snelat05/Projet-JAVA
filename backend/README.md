# appmanagement

Backend du POS (Point of Sale) — V2 du systeme Auto-Parts Management.
Spring Boot 4 / Java 21 / PostgreSQL 16.

> Cible V1 : Haiti / Caraibes. Multi-devises (HTG / USD), multilingue (FR / Creole / EN), hors-ligne via PWA cote frontend.

## Prerequis

- **Java 21** — `java --version`
- **Docker + Docker Compose** — `docker --version`
- **Make** (deja present sur Linux/macOS/WSL)
- **Terminal Unix** : lance les commandes depuis WSL Ubuntu, pas Windows cmd/PowerShell

## Demarrage rapide

```bash
make setup     # cree .env a partir de .env.example
make up        # demarre Postgres + pgAdmin via Docker
make run       # lance l'application Spring Boot
```

Ou tout en une commande : `make dev`.

## Acces aux services

| Service | URL |
|---|---|
| API REST | http://localhost:8080 |
| OpenAPI / Swagger UI | http://localhost:8080/swagger-ui.html |
| Actuator (health) | http://localhost:8080/actuator/health |
| pgAdmin | http://localhost:5050 |
| PostgreSQL | localhost:5432 |

Credentials pgAdmin et Postgres : voir `.env` (non versionne).

## Commandes Make

Tape `make` ou `make help` pour la liste complete.

| Commande | Description |
|---|---|
| `make setup` | Premiere installation (cree `.env`) |
| `make up` / `make down` | Demarre / arrete les conteneurs |
| `make restart` | Redemarre les conteneurs |
| `make logs` | Suit les logs Docker |
| `make ps` | Etat des conteneurs |
| `make psql` | Ouvre psql dans le conteneur Postgres |
| `make db-reset` | **DESTRUCTIF** — supprime le volume DB et recree |
| `make run` | Lance Spring Boot (auto-migration Flyway) |
| `make dev` | `up` + `run` enchaines |
| `make build` | Compile en JAR (sans tests) |
| `make test` | Lance les tests |
| `make clean` | Nettoie `target/` |

## Stack technique

| Couche | Technologie |
|---|---|
| Backend | Spring Boot 4.0.6 + Java 21 |
| Auth | Spring Security + JWT (jjwt 0.12.3) |
| ORM | Spring Data JPA + Hibernate |
| Migrations | Flyway (`src/main/resources/db/migration/`) |
| DTO mapping | MapStruct 1.5.5 |
| Doc API | SpringDoc OpenAPI 2.3.0 |
| DB | PostgreSQL 16 |
| Frontend (a venir) | React + TypeScript + Vite + PWA |

## Conventions importantes

- **Migrations** : tout changement de schema passe par un fichier Flyway `V{n}__{description}.sql` dans `src/main/resources/db/migration/`. Jamais de modification manuelle.
- **`ddl-auto: validate`** : Hibernate ne touche jamais le schema. Flyway pilote tout. Si l'app refuse de demarrer, c'est qu'une entite ne correspond pas a une migration.
- **Multi-devises** : tout montant en `BigDecimal` avec colonne `currency` et reference au taux historise. Jamais de `double`/`float` pour de l'argent.
- **Soft delete** : colonne `is_active BOOLEAN DEFAULT TRUE`. Pas de `DELETE` physique.
- **Pagination obligatoire** : tout endpoint qui retourne une liste accepte `Pageable`.
- **Validation** : `@Valid` + Bean Validation sur tous les DTO entrants.
- **Erreurs** : `@ControllerAdvice` + `ProblemDetail` (RFC 7807). Jamais exposer un message d'exception DB au client.

## Workflow Git

| Branche | Usage |
|---|---|
| `main` | Production stable — jamais de push direct |
| `develop` | Integration des features |
| `feature/xxx` | Une feature par branche, PR vers `develop` |
| `hotfix/xxx` | Correction urgente prod — merge dans `main` + `develop` |

Pull Request obligatoire pour merger, minimum 1 review.

Voir [`docs/git-workflow.md`](docs/git-workflow.md) pour le document complet et [`docs/git-cheatsheet.md`](docs/git-cheatsheet.md) pour les commandes quotidiennes.

## Variables d'environnement

Toute la configuration sensible est dans `.env` (gitignored). Voir `.env.example` pour la liste et les valeurs par defaut.

## Decisions techniques

Les decisions structurantes du projet sont documentees dans [`docs/adr/`](docs/adr/) (Architecture Decision Records).

| ADR | Sujet | Statut |
|---|---|---|
| [0001](docs/adr/0001-stack-technique.md) | Stack technique : Spring Boot + React + Postgres | Accepted |
| [0002](docs/adr/0002-pwa-first.md) | PWA d'abord, pas d'app native en V1 | Accepted |
| [0003](docs/adr/0003-offline-niveau-b.md) | Hors-ligne : queue + sync, un seul caissier offline | Accepted |

Lire [`docs/adr/README.md`](docs/adr/README.md) pour comprendre le format et comment proposer une nouvelle ADR.

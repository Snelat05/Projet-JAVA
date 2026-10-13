.DEFAULT_GOAL := help

# Auto-charge .env si present (les variables sont exposees aux sous-commandes)
ifneq (,$(wildcard ./.env))
    include .env
    export
endif

.PHONY: help setup up down restart logs ps psql db-reset build run dev test clean

help: ## Affiche cette aide
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

setup: ## Premiere installation — copie .env.example vers .env si absent
	@test -f .env || cp .env.example .env
	@echo "OK — .env pret. Lance ensuite 'make up' puis 'make run'."

# === Docker (Postgres + pgAdmin) ===

up: ## Demarre Postgres + pgAdmin en arriere-plan
	docker compose up -d

down: ## Arrete les conteneurs (les donnees du volume sont conservees)
	docker compose down

restart: down up ## Redemarre les conteneurs

logs: ## Suit les logs des conteneurs Docker
	docker compose logs -f

ps: ## Etat des conteneurs
	docker compose ps

# === Base de donnees ===

psql: ## Ouvre une session psql dans le conteneur Postgres
	docker compose exec postgres psql -U $(POSTGRES_USER) -d $(POSTGRES_DB)

db-reset: ## DESTRUCTIF — supprime le volume Postgres et recree la DB
	@echo "ATTENTION : toutes les donnees vont etre supprimees."
	@read -p "Confirmer (y/N) : " ans && [ "$$ans" = "y" ] || exit 1
	docker compose down -v
	docker compose up -d

# === Spring Boot ===

build: ## Compile et package en JAR (sans tests)
	./mvnw clean package -DskipTests

run: ## Lance l'application Spring Boot (les migrations Flyway s'appliquent au demarrage)
	./mvnw spring-boot:run

dev: up run ## Demarre Docker puis lance l'app (workflow le plus courant)

test: ## Execute la suite de tests
	./mvnw test

clean: ## Nettoie le dossier target/
	./mvnw clean

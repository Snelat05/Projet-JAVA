# ADR-0001 : Stack technique : Spring Boot + React + PostgreSQL

- **Statut** : Accepted
- **Date** : 2026-05-25
- **Decideurs** : Lead technique
- **Tags** : `backend`, `frontend`, `db`, `fondation`

## Contexte

Le projet est la V2 d'un systeme Auto-Parts Management ecrit en PHP. Un audit technique senior d'avril 2026 a identifie 12 dettes critiques sur la V1 (credentials en dur, pas de CSRF, CORS ouvert, session sans timeout, logique metier dans Controllers, requetes N+1, pas de pagination, pas de fichier `.env`...). Score securite : 3/10.

On rebatit la V2 a partir d'une page blanche, avec :
- Une equipe de **4 developpeurs juniors**
- Une cible marche **Haiti / Caraibes** (connectivite instable, multi-devises HTG/USD, multilingue)
- Un MVP a livrer en **3 mois** : auth + roles, produits CRUD, stock, POS, dashboard
- Un besoin de **pouvoir justifier** chaque choix face a l'equipe et aux parties prenantes

## Decision drivers

- Securite par defaut (eviter les dettes de la V1)
- Courbe d'apprentissage acceptable pour une equipe junior
- Ecosysteme mature avec beaucoup de ressources d'apprentissage
- Marche de l'emploi pour valoriser l'experience plus tard
- Bundle frontend leger (cible : < 200 Ko gzip en initial load) pour le contexte 3G/4G Haiti
- Compatibilite multi-OS sans refaire l'application

## Options considerees

### Option A : Garder PHP, migrer vers Laravel

- **Description** : refacto progressif du code PHP existant vers Laravel.
- **Pour** : continuite, equipe deja familiere avec PHP, l'audit V1 suggerait cette voie.
- **Contre** : on garde une partie de la dette mentale du code V1 ; Spring Security est largement plus robuste que ce qu'on peut bricoler en PHP ; ecosysteme JavaScript natif manquant pour le PWA hors-ligne.

### Option B : Node.js (NestJS) + React

- **Description** : backend Node/NestJS, frontend React.
- **Pour** : un seul langage (JS/TS) sur toute la stack ; iteration rapide.
- **Contre** : NestJS est plus "magique" que Spring Boot et moins discute en francais ; performances inferieures pour les operations transactionnelles (POS) ; ecosysteme securite/auth moins mature que Spring Security.

### Option C : Spring Boot + React + PostgreSQL (choisi)

- **Description** : backend Java 21 / Spring Boot 4, frontend React + TypeScript, DB PostgreSQL.
- **Pour** : Spring Security tres mature et opinionated (force a bien faire les choses) ; Spring Data JPA simplifie le DAO ; React est le standard de l'industrie cote front ; PostgreSQL est solide pour transactions financieres.
- **Contre** : Spring Boot est verbeux ; courbe d'apprentissage initiale plus raide pour des juniors ; hot reload moins agreable qu'avec Node.

### Option D : .NET 8 + Blazor

- **Description** : full stack Microsoft.
- **Pour** : tres bonne integration, performances excellentes.
- **Contre** : moins repandu au Quebec / Caraibes que Java ; equipe non familiere ; Blazor encore considere comme "nouveau" pour PWA serieux.

## Decision

On choisit **Option C : Spring Boot 4.0.6 + Java 21 + React + TypeScript + PostgreSQL 16**.

Raisons principales :
- **Securite par defaut** : Spring Security avec CSRF active, gestion des sessions, JWT correctement implementable. C'est exactement ce qu'il faut pour ne pas reproduire les dettes de la V1 (driver #1).
- **Courbe d'apprentissage acceptable** : Java est largement enseigne dans les programmes universitaires que l'equipe a suivis. Spring Boot a une documentation francophone abondante (driver #2).
- **Ecosysteme mature** : MapStruct, Flyway, SpringDoc OpenAPI, Lombok existent depuis plus de 10 ans et sont stables (driver #3).
- **Marche de l'emploi** : combinaison Spring + React parmi les plus recherchees au Quebec (driver #4).
- **Bundle leger possible** : React + Vite + Tailwind + shadcn/ui permet un bundle < 200 Ko gzip avec tree-shaking. PHP cote front n'aurait pas permis ca (driver #5).
- **PWA pour multi-OS** : React + Service Worker = installable sur tous les OS sans refaire l'app (driver #6).

## Consequences

### Positives

- Stack durable et reconnaissable sur un CV
- Securite robuste sans avoir a tout coder soi-meme
- Documentation API automatique via SpringDoc OpenAPI
- Migrations DB versionnees via Flyway
- Possibilite de scaler vers une architecture micro-services plus tard si besoin

### Negatives

- **Verbosite Java** : plus de code a ecrire qu'en TypeScript ou Python pour le meme resultat
- **Demarrage application lent** : Spring Boot prend 10-20 secondes a demarrer (pas un probleme en prod, ennuyeux en dev). Mitigation : utiliser DevTools (deja en dependance) pour le hot reload.
- **Courbe d'apprentissage** : Spring Security est puissant mais complexe. Premier sprint = formation collective necessaire. Mitigation : pair programming sur la feature auth.
- **Deux langages** : Java cote back, TypeScript cote front. Mitigation : MapStruct genere les DTO, et OpenAPI peut generer les clients TypeScript automatiquement.

### Mitigation

- Premiere semaine = setup + tutoriel collectif Spring Boot
- ADR-002 et ADR-003 completent ce choix pour le frontend (PWA, offline)
- Pair programming systematique pour le sprint 1

## Liens

- ADR-0002 : PWA d'abord, pas d'app native
- ADR-0003 : Offline niveau B
- Audit V1 : `docs/audit_v1.pdf` (externe)
- [Spring Boot 4 reference](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [React + TypeScript handbook](https://react.dev/learn/typescript)

# Workflow Git du projet appmanagement

Document de reference. **A lire avant le premier commit.**

> Pour un usage quotidien rapide, voir [`git-cheatsheet.md`](git-cheatsheet.md).

## Pourquoi un workflow strict ?

On est 4 developpeurs sur un meme repo. Sans regles claires, on va se marcher sur les pieds, ecraser le travail des autres, casser la prod, et perdre du temps a chercher qui a fait quoi.

Le workflow ci-dessous est **simple mais non-negociable** :
- Personne ne push directement sur `main` ou `develop`
- Tout changement passe par une **Pull Request** (PR) validee par au moins 1 autre dev
- Les messages de commit suivent une convention unique

## Vue d'ensemble

```
   main          ●──────●──────●──────●           <- production (versions taggees)
                 ↑      ↑      ↑      ↑
   develop  ●─●─●──●─●─●──●─●─●──●─●─●            <- integration
            ↑    ↑    ↑    ↑    ↑    ↑
   feature/auth ─┘    │    │    │    │
   feature/products ──┘    │    │    │
   feature/stock ──────────┘    │    │
   fix/login-error ─────────────┘    │
   hotfix/payment-bug ───────────────┘
```

- **Toutes les features** branchent depuis `develop`, retournent dans `develop`
- **Les releases** : on merge `develop` -> `main` et on tag (`v0.1.0`, `v0.2.0`, etc.)
- **Les hotfix** (urgents prod) : ils branchent de `main`, retournent dans `main` ET `develop`

## Les 4 types de branches

### `main` — production

- Toujours **deployable**
- **Jamais de push direct** (techniquement bloque)
- Seules les PRs venant de `develop` ou `hotfix/*` peuvent y entrer
- Chaque merge dans `main` doit etre tagge (`git tag v0.1.0`)

### `develop` — integration

- C'est ici que toutes les features sont assemblees avant une release
- **Jamais de push direct** (techniquement bloque)
- Seules les PRs venant de `feature/*`, `fix/*` ou `docs/*` peuvent y entrer

### `feature/<description>` — nouvelle fonctionnalite

- Branche **depuis `develop`**, PR **vers `develop`**
- Nommage : `feature/auth-jwt`, `feature/products-crud`, `feature/sales-checkout`
- Une feature = un ticket = une PR. Ne PAS mettre 3 features dans une seule branche.
- Duree de vie courte : 1 a 5 jours max. Au-dela, c'est trop gros, decoupe.

### `hotfix/<description>` — correction urgente en production

- Branche **depuis `main`**, PR **vers `main` ET `develop`**
- Reserve aux bugs critiques en prod (le POS ne peut plus encaisser, par exemple)
- Apres le merge dans `main`, on tag (`v0.1.1`)

### Autres prefixes acceptes

- `fix/xxx` — correction non urgente (PR vers `develop`)
- `docs/xxx` — documentation seule (PR vers `develop`)
- `refactor/xxx` — refacto sans changement fonctionnel (PR vers `develop`)
- `chore/xxx` — maintenance (dependances, config) (PR vers `develop`)

## Convention de commit (Conventional Commits)

Tous les commits suivent ce format :

```
<type>(<scope>): <description courte en francais>

[corps optionnel : pourquoi, contexte]

[footer optionnel : refs issues, breaking changes]
```

### Types acceptes

| Type | Quand l'utiliser |
|---|---|
| `feat` | Nouvelle fonctionnalite visible utilisateur |
| `fix` | Correction de bug |
| `docs` | Documentation seule (README, ADR, commentaires) |
| `style` | Formatage, indentation, espaces (pas de changement de logique) |
| `refactor` | Refacto sans changement fonctionnel ni correction |
| `test` | Ajout ou modification de tests |
| `chore` | Maintenance, dependances, config build |
| `ci` | Pipeline CI/CD (GitHub Actions) |

### Scope (optionnel mais recommande)

Le module ou la feature concerne : `auth`, `products`, `stock`, `sales`, `stores`, `users`, `common`, `db`.

### Exemples valides

```
feat(auth): ajouter endpoint login JWT
fix(products): corriger validation prix negatif
docs(adr): ajouter ADR-0004 multi-devises
refactor(stock): extraire StockMovementService depuis ProductService
test(sales): ajouter cas limite caisse fermee
chore(deps): bump spring-boot 4.0.6 -> 4.0.7
ci: ajouter workflow tests sur PR
```

### Exemples invalides

```
update                          # type manquant, pas descriptif
fix bug                         # quel bug ?
WIP                             # pas un commit final, ne devrait pas etre push
fix: it works now              # quoi "it" ?
```

### Pourquoi cette convention ?

- L'historique reste **lisible** apres 1000 commits
- On peut **filtrer** facilement : `git log --grep "feat(auth)"` montre tout ce qui touche l'auth
- On peut **generer automatiquement** un CHANGELOG quand on tag une release
- Les outils CI peuvent **bloquer** les commits mal formates

## Workflow complet d'une feature

### 1. Avant de commencer

```bash
git checkout develop
git pull origin develop      # toujours partir de la derniere version
git checkout -b feature/products-crud
```

### 2. Pendant le developpement

```bash
# Commits frequents et petits
git add fichier1 fichier2
git commit -m "feat(products): ajouter entite Product"

git add fichier3
git commit -m "feat(products): ajouter ProductRepository"

# Push regulier pour ne pas perdre son travail
git push origin feature/products-crud
```

**Regle d'or** : commit au moins **une fois par jour**, meme si c'est incomplet (statut WIP local) — mais **ne push** que ce qui compile.

### 3. Synchroniser avec develop pendant le dev

Si la feature dure plus de 2 jours, tu dois recuperer les changements de `develop` regulierement pour eviter un gros conflit a la fin :

```bash
git checkout develop
git pull origin develop
git checkout feature/products-crud
git merge develop            # ou git rebase develop (voir ci-dessous)
```

### 4. Ouvrir la Pull Request

Une fois la feature prete :

```bash
git push origin feature/products-crud
```

Puis sur GitHub :
1. Clic sur "Compare & pull request"
2. Verifier que la cible est **`develop`** (pas `main`)
3. Remplir le template (description, comment tester, checklist)
4. Assigner **au moins 1 reviewer**
5. Attendre la review

### 5. Apres la review

- Si "Changes requested" : corriger, commit, push -> la PR se met a jour automatiquement
- Si "Approved" : merge possible. Utiliser **"Squash and merge"** pour garder un historique propre dans `develop`

### 6. Apres le merge

```bash
git checkout develop
git pull origin develop
git branch -d feature/products-crud       # supprime la branche locale
```

GitHub supprime automatiquement la branche distante (configure dans les settings du repo).

## Merge vs Rebase — la regle simple

Pour les juniors, **regle a respecter** :

| Situation | Quoi faire |
|---|---|
| Synchroniser ma branche feature avec develop | `git merge develop` (plus simple, moins de risques) |
| Nettoyer mes commits avant la PR | Optionnel, on s'en occupe avec "Squash and merge" |
| Merger ma PR dans develop | **Squash and merge** depuis GitHub UI |
| Merger develop dans main | **Merge commit** (pour garder l'historique des features) |

**Ne JAMAIS** :
- `git push --force` sur une branche partagee (peut effacer le travail des autres)
- Rebaser une branche que quelqu'un d'autre a deja recuperee
- Faire un rebase interactif sans demander d'abord

## Regles non-negociables

1. **Aucun push direct sur `main` ou `develop`** (bloque par GitHub)
2. **Aucune PR mergeable sans review** (minimum 1 approbation)
3. **Aucune PR mergeable si la CI echoue** (tests rouges = pas de merge)
4. **Aucun commit de `.env`, `node_modules`, `target/`, secrets** (gitignore + revue PR)
5. **Aucun `git push --force` sur `main` ou `develop`** (bloque par GitHub)
6. **Aucune branche `feature/` qui vit plus de 7 jours** sans synchronisation avec `develop`
7. **Un commit, une intention** (pas 12 changements melanges dans un commit "update")

## Branch protection GitHub a configurer

A faire **avant le premier sprint** par le lead, dans Settings > Branches du repo :

### Pour `main`

- [x] Require a pull request before merging
- [x] Require approvals (1 minimum)
- [x] Dismiss stale approvals when new commits are pushed
- [x] Require status checks to pass before merging (cocher les jobs CI quand ils existent)
- [x] Require branches to be up to date before merging
- [x] Restrict who can push to matching branches (vide = personne, parfait)
- [x] Do not allow bypassing the above settings

### Pour `develop`

Memes regles que `main`, plus relaches si necessaire (par exemple "require approvals" peut etre 1 au lieu de 2).

## Que faire en cas de conflit

Pas de panique. Un conflit Git c'est normal, ca veut dire que deux personnes ont modifie le meme fichier au meme endroit. Voici comment proceder :

```bash
# Sur ta branche feature
git checkout feature/ma-feature
git pull origin develop      # ou git merge origin/develop

# Git affiche les fichiers en conflit
# Ouvre-les, tu vas voir des marqueurs :
#
#   <<<<<<< HEAD
#   ton code
#   =======
#   le code de l'autre
#   >>>>>>> develop
#
# Choisis quoi garder (peut-etre les deux, dans le bon ordre)
# Supprime les marqueurs <<<<, ====, >>>>

git add fichier-resolu
git commit -m "merge: resoudre conflit avec develop"
git push origin feature/ma-feature
```

**Si tu n'es pas sur** : **demande de l'aide AVANT de commit le merge**. Un conflit mal resolu peut effacer du code silencieusement.

## Formation initiale (atelier 1h30)

Avant le sprint 1, faire un atelier pratique avec toute l'equipe :

| Duree | Sujet |
|---|---|
| 15 min | Survol du workflow (ce document) |
| 15 min | Configuration locale (SSH, gitconfig, alias) |
| 20 min | Demo en live : creer une feature, commit, PR, merge |
| 20 min | Exercice : chacun cree une branche `feature/intro-<prenom>`, ajoute un fichier dans `docs/team/`, ouvre une PR, review celle d'un autre |
| 10 min | Demo gestion de conflit (le lead provoque un conflit volontaire) |
| 10 min | Q&A |

## Ressources

- [Pro Git book (FR)](https://git-scm.com/book/fr/v2) — chapitres 3 (branches) et 5 (workflows)
- [Conventional Commits](https://www.conventionalcommits.org/fr/)
- [GitHub Flow vs GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [Oh Shit, Git!?!](https://ohshitgit.com/fr) — comment recuperer apres une erreur

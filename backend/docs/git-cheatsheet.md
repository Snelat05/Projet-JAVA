# Git — Antiseche quotidienne

Pour le workflow complet, voir [`git-workflow.md`](git-workflow.md).
Cette page = les commandes du quotidien, a garder ouverte.

## Setup initial (une seule fois)

```bash
# Configurer son identite
git config --global user.name "Prenom Nom"
git config --global user.email "ton.email@example.com"

# Cloner le repo
git clone git@github.com:<org>/appmanagement.git
cd appmanagement
make setup        # cree .env

# Verifier la branche par defaut
git checkout develop
```

## Commencer une nouvelle feature

```bash
git checkout develop
git pull origin develop                       # toujours partir a jour
git checkout -b feature/ma-feature            # creer la branche
```

## Pendant le dev

```bash
git status                                    # ou j'en suis
git diff                                      # ce que j'ai change
git add fichier1 fichier2                     # stager des fichiers precis
git add -p                                    # stager interactivement (recommande)
git commit -m "feat(scope): description"      # committer
git push origin feature/ma-feature            # pousser (1ere fois)
git push                                      # pousser (apres)
```

## Recuperer le travail des autres

```bash
git fetch origin                              # voir ce qui a change sur le serveur
git pull origin develop                       # quand tu es sur develop
git pull                                      # raccourci si la branche est trackee
```

## Synchroniser sa branche feature avec develop

```bash
git checkout develop
git pull origin develop
git checkout feature/ma-feature
git merge develop                             # integre develop dans ma branche
# Resoudre les conflits eventuels, puis :
git push
```

## Ouvrir une Pull Request

```bash
git push origin feature/ma-feature
# Puis aller sur GitHub : "Compare & pull request"
# Cible : develop (pas main)
# Remplir le template
# Assigner un reviewer
```

## Annuler des choses (les commandes qui sauvent)

### Annuler des changements pas encore stages

```bash
git restore fichier                           # annule les modifs locales
git restore .                                 # annule TOUT (attention)
```

### Annuler un fichier stage (mis avec git add)

```bash
git restore --staged fichier                  # le retire de l'index
```

### Modifier le dernier commit (pas encore push)

```bash
git commit --amend -m "nouveau message"       # change le message
git add fichier-oublie
git commit --amend --no-edit                  # ajoute un fichier au dernier commit
```

> **JAMAIS** d'amend sur un commit deja push partage.

### Revenir a un commit precedent (sans le perdre)

```bash
git log --oneline                             # trouver le hash
git checkout <hash>                           # se met en mode "detached HEAD"
# Pour revenir au present :
git checkout develop
```

### J'ai commit sur la mauvaise branche

```bash
# J'ai commit sur develop par erreur, je voulais une feature branch :
git branch feature/mon-truc                   # cree la branche au point actuel
git reset --hard origin/develop               # remet develop a l'etat distant
git checkout feature/mon-truc                 # ma feature contient mon travail
```

### Stash : mettre de cote temporairement

```bash
git stash                                     # met de cote tout le travail non commit
git stash pop                                 # le remet
git stash list                                # voir les stashes
```

## Gerer un conflit

```bash
# Git te le dit dans git status apres un pull/merge
git status                                    # liste les fichiers en conflit

# Ouvrir chaque fichier, chercher les marqueurs :
#   <<<<<<< HEAD
#   ton code
#   =======
#   le code de l'autre
#   >>>>>>> develop
# Choisir, supprimer les marqueurs

git add fichier-resolu
git commit                                    # message auto-genere par Git
git push
```

> **Si tu doutes** : NE COMMIT PAS. Demande au lead. Un conflit mal resolu peut effacer du code sans alerter.

## Voir l'historique

```bash
git log --oneline -10                         # 10 derniers commits, format compact
git log --graph --oneline --all -20           # graph visuel
git log --author="Prenom"                     # commits d'une personne
git log --grep="auth"                         # commits qui mentionnent "auth"
git show <hash>                               # detail d'un commit
git blame fichier                             # qui a ecrit chaque ligne
```

## Apres le merge de ma PR

```bash
git checkout develop
git pull origin develop                       # recupere mon merge
git branch -d feature/ma-feature              # supprime la branche locale
# La branche distante est supprimee automatiquement par GitHub
```

## Alias utiles a mettre dans ~/.gitconfig

```ini
[alias]
    st = status
    co = checkout
    br = branch
    cm = commit -m
    lg = log --graph --oneline --all -20
    sync = !git checkout develop && git pull origin develop
    unstage = restore --staged
```

Utilisation : `git st`, `git lg`, `git sync`, etc.

## Commandes interdites

| Commande | Pourquoi |
|---|---|
| `git push --force` sur main/develop | Efface le travail des autres |
| `git rebase` sur une branche partagee | Refait l'historique, casse les autres |
| `git push --force-with-lease` sans comprendre | Idem |
| `git reset --hard` sans avoir push avant | Tu perds ton travail |
| `git clean -fd` sans verifier | Supprime tes fichiers non trackes |

## Quand tout part en vrac

1. **NE PAS PANIQUER**
2. **NE PAS faire de `--force`, `reset --hard`, `clean -fd`**
3. Sauvegarder ton travail en cours : `git stash` ou copie manuelle des fichiers
4. Demander de l'aide au lead

Git garde une **trace de tout pendant 90 jours** dans le reflog. Presque rien n'est vraiment perdu :

```bash
git reflog                                    # historique de TOUTES tes operations locales
git checkout <hash-du-reflog>                 # retrouver un etat perdu
```

## Quand demander de l'aide

Toujours demander si :
- Tu vois "diverged" dans `git status`
- Tu as un conflit sur plus de 3 fichiers
- Tu as fait un `reset` ou un `rebase` et tu n'es pas sur du resultat
- Tu vois le mot "HEAD" sans comprendre
- Tu hesites a taper `--force`

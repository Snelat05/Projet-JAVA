# ADR-0003 : Hors-ligne niveau B — queue locale + sync, un seul caissier offline par magasin

- **Statut** : Accepted
- **Date** : 2026-05-25
- **Decideurs** : Lead technique
- **Tags** : `frontend`, `offline`, `sync`, `architecture`

## Contexte

La cible marche V1 est **Haiti / Caraibes**, ou la connectivite Internet est instable :
- 3G/4G frequemment intermittente
- Coupures de courant qui interrompent le wifi
- Zones rurales avec couverture limitee
- Pics de demande qui saturent les operateurs

Le POS **doit continuer a vendre** meme sans Internet. Sinon le commercant ne peut pas l'adopter.

Mais le hors-ligne est un domaine **technique tres difficile** : synchroniser plusieurs clients qui modifient les memes donnees simultanement = un des problemes les plus durs en systeme distribue.

Avec une equipe junior, on doit choisir un niveau **realiste**.

## Decision drivers

- Time-to-market V1 : 3 mois
- Niveau d'experience equipe : aucun n'a deja construit de systeme distribue
- Cas d'usage reel : la plupart des commercants en Haiti ont 1 caisse, pas 5
- Risque metier : si la sync echoue, c'est de l'argent (transactions perdues)
- Maintenance : le code doit etre comprehensible par les juniors qui vont l'entretenir

## Options considerees

### Option A : Lecture seule hors-ligne

- **Description** : on cache le catalogue produit en IndexedDB pour pouvoir consulter le stock offline, mais toute ecriture (vente, mouvement stock) necessite la connexion.
- **Pour** : simple a implementer, aucun risque de conflit.
- **Contre** : ne resout pas le probleme principal — vendre hors-ligne. Inutile pour un POS.

### Option B : Queue d'ecriture locale + sync, **un seul caissier offline par magasin** (choisi)

- **Description** : toutes les operations offline (ventes, mouvements stock) sont enfilees dans une queue locale IndexedDB. A la reconnexion, la queue est envoyee au serveur en ordre. **Limitation acceptee** : si deux caissiers d'un meme magasin sont offline simultanement, la deuxieme reconnexion peut detecter des conflits qui devront etre resolus manuellement.
- **Pour** : couvre 90% des cas reels (un commerce, une caisse, perte de connexion ponctuelle) ; implementable en 1 sprint dedie ; testable.
- **Contre** : ne supporte pas le scenario "3 caisses offline en meme temps qui vendent le meme produit" ; necessite de communiquer cette limitation aux clients.

### Option C : Multi-master avec CRDT (Conflict-free Replicated Data Types)

- **Description** : utiliser des structures de donnees mathematiquement convergentes (G-Counter, OR-Set, etc.) qui resolvent automatiquement les conflits.
- **Pour** : solution theoriquement parfaite ; supporte N caissiers offline simultanement.
- **Contre** : domaine de recherche academique recent ; **aucun membre de l'equipe ne maitrise** ; 3 a 6 mois de R&D ; ecosysteme JS encore immature (Yjs, Automerge sont jeunes) ; un bug = des transactions financieres incoherentes = catastrophe.

## Decision

On choisit **Option B : Niveau B** pour la V1.

Limitation explicite : **un seul caissier peut etre offline a la fois par magasin**. Si un deuxieme caissier perd la connexion alors qu'un premier est deja offline, le deuxieme est bloque ou en mode lecture seule.

## Consequences

### Positives

- Implementable en 1 sprint dedie par 2 devs
- Code lisible et maintenable par des juniors
- Couvre le cas d'usage reel (la grande majorite des commerces en Haiti ont 1 caisse)
- Permet une experience "POS qui marche tout le temps" pour le caissier principal
- Architecture compatible avec une evolution future vers le niveau C (si vraiment necessaire)

### Negatives

- **Limitation business** : impossible de vendre depuis 3 caisses en simultane sans connexion. **Mitigation** : la documenter clairement dans le contrat client et dans l'app (badge "Hors-ligne — un seul caissier autorise").
- **Conflits residuels** : meme avec un seul caissier offline, des conflits peuvent survenir si le stock a ete modifie cote serveur entre temps. **Mitigation** : detecter les conflits a la sync et presenter une UI de resolution manuelle.
- **Risque de perte** : si IndexedDB est corrompue ou effacee avant sync, les transactions sont perdues. **Mitigation** : sync agressive (toutes les 30 secondes si connexion presente) + backup local sur un deuxieme storage (localStorage en miroir des ids de transactions critiques).

### Mitigation principale

Si en production on observe que des clients ont reellement besoin du niveau C (3+ caisses simultanees offline), on ouvre une nouvelle ADR pour evaluer la migration. **Pas avant.**

## Stack technique pour implementer le niveau B

- **IndexedDB** via [Dexie.js](https://dexie.org/) pour le stockage local structure
- **Service Worker** pour intercepter les requetes API quand offline et les enfiler
- **TanStack Query** avec [persistance offline](https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient) pour le cache des GET
- **Background Sync API** (la ou supportee) pour declencher la sync au retour de la connexion
- Cote backend : **endpoints idempotents** (chaque transaction porte un UUID client genere offline, le serveur ignore les doublons)

## Liens

- ADR-0001 : Stack technique
- ADR-0002 : PWA d'abord
- [Dexie.js documentation](https://dexie.org/)
- [Background Sync API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API)
- [Pourquoi CRDT est dur](https://josephg.com/blog/crdts-are-the-future/) — lecture optionnelle pour comprendre pourquoi on a refuse l'option C

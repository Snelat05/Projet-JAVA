# Architecture Decision Records (ADR)

Ce dossier contient les **decisions techniques importantes** prises sur le projet.
Chaque ADR documente **pourquoi** une decision a ete prise, pas seulement ce qui a ete decide.

## Pourquoi des ADR ?

Quand un membre de l'equipe demande dans 6 mois *« pourquoi on a choisi Spring Boot et pas Node.js ? »*,
la reponse doit etre dans une ADR — pas dans un Slack qu'on ne retrouvera jamais, pas dans la tete du lead.

Les ADR remplissent 3 roles :
1. **Justifier** une decision avec ses alternatives et ses tradeoffs
2. **Onboarder** un nouveau dev rapidement (il lit les ADR pour comprendre le projet)
3. **Re-evaluer** une decision si le contexte change

## Format

On utilise **MADR** (Markdown Architecture Decision Records), version simplifiee.
Chaque ADR contient :

- Un **statut** : `Accepted`, `Superseded`, `Deprecated`
- Le **contexte** : quel probleme on resout
- Les **options considerees** : alternatives evaluees (jamais une seule option)
- La **decision** : ce qu'on a choisi et pourquoi
- Les **consequences** : positives ET negatives

Voir [`template.md`](template.md) pour le gabarit.

## Convention de nommage

```
docs/adr/NNNN-titre-en-kebab-case.md
```

- `NNNN` : numero a 4 chiffres, incremente (`0001`, `0002`, ...)
- Le numero ne se reutilise jamais, meme si une ADR est supprimee

## Comment proposer une nouvelle ADR

1. Copie `template.md` vers `docs/adr/NNNN-mon-titre.md`
2. Remplis les sections (10-20 min de travail)
3. Pousse sur une branche `feature/adr-NNNN-mon-titre`
4. Ouvre une PR — l'ADR doit etre **discutee et validee** comme du code
5. Une fois mergee avec statut `Accepted`, elle fait loi

## Comment remplacer une ADR existante

On ne **modifie jamais** une ADR `Accepted`. On en ecrit une nouvelle avec statut `Accepted`
et on met l'ancienne en statut `Superseded by ADR-NNNN`.

Ca permet de garder la trace historique des choix, meme quand ils changent.

## Liste des ADR

| Numero | Titre | Statut |
|---|---|---|
| [0001](0001-stack-technique.md) | Stack technique : Spring Boot + React + Postgres | Accepted |
| [0002](0002-pwa-first.md) | PWA d'abord, pas d'app native en V1 | Accepted |
| [0003](0003-offline-niveau-b.md) | Hors-ligne : queue locale + sync, un seul caissier offline | Accepted |

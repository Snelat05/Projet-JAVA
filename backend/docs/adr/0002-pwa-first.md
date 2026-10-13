# ADR-0002 : PWA d'abord, pas d'app native en V1

- **Statut** : Accepted
- **Date** : 2026-05-25
- **Decideurs** : Lead technique
- **Tags** : `frontend`, `mobile`, `deploiement`

## Contexte

Le POS doit etre utilisable sur **plusieurs supports** : ordinateurs de bureau (caisse fixe), tablettes (POS mobile dans le magasin), smartphones (consultation hors site), Windows / Android / iOS / Linux.

Le resume de la reunion initiale evoquait pele-mele : Electron, Capacitor, applications natives iOS/Android, applications hybrides. Aucune decision n'avait ete prise.

Avec une equipe de **4 developpeurs juniors** et un MVP a livrer en **3 mois**, on doit choisir une approche qui :
- Maximise la couverture des plateformes avec un effort minimal
- Ne demande pas d'expertise mobile native (que l'equipe n'a pas)
- Permette des mises a jour rapides sans passer par des stores

## Decision drivers

- **Time-to-market** : V1 livree en 3 mois
- **Effort de developpement** : un seul code base, pas plusieurs
- **Mises a jour** : possibilite de deployer plusieurs fois par semaine sans review d'Apple/Google
- **Hors-ligne** : critique pour Haiti (voir ADR-0003)
- **Installation** : l'utilisateur doit pouvoir "installer" l'app sur son appareil
- **Acces materiel** : imprimante thermique, scanner code-barres (la plupart sont HID / WebUSB / WebBluetooth)

## Options considerees

### Option A : Applications natives separees (iOS Swift + Android Kotlin + desktop Electron)

- **Description** : 3 codebases distincts, expertises distinctes.
- **Pour** : meilleures performances natives, acces complet aux APIs systeme.
- **Contre** : 3x plus de travail ; aucun dev de l'equipe n'a d'experience iOS/Swift ou Android/Kotlin avancee ; cycle de mise a jour bloque par App Store (24-48h) et Play Store.

### Option B : Hybride avec Capacitor (V1)

- **Description** : un seul code React, wrapping avec Capacitor pour generer des binaires iOS et Android.
- **Pour** : un seul code base ; acces a quelques APIs natives via plugins ; presence dans les stores.
- **Contre** : il faut gerer les builds iOS (Mac requis, compte Apple Developer 99 $/an) et Android ; complexite des stores pour une V1 ; plus lourd en demarrage qu'une PWA.

### Option C : Electron pour desktop

- **Description** : packager le frontend dans un Chromium embarque pour Windows/Mac/Linux.
- **Pour** : application "desktop" installable.
- **Contre** : 150-200 Mo par installation (un Chromium complet) ; ne resout pas le probleme mobile ; PWA fait deja la meme chose en mieux.

### Option D : PWA (Progressive Web App) (choisi)

- **Description** : application web React avec Service Worker, manifest, et stockage offline. Installable depuis le navigateur sur Windows / Mac / Linux / Android / iOS (avec quelques limites).
- **Pour** : un seul code base, un seul deploiement, aucun store, mises a jour immediates, fonctionne hors-ligne via Service Worker.
- **Contre** : limitations iOS Safari sur certaines APIs (Bluetooth, notifications push) ; perception "moins serieux" qu'une app native dans les stores ; certains utilisateurs ne savent pas comment installer une PWA.

## Decision

On choisit **Option D : PWA en V1**.

Capacitor (Option B) reste une porte de sortie possible **en V2 si necessaire** : le code React reste identique, on rajoute juste Capacitor par-dessus.

Raisons principales :
- **Time-to-market** : zero travail mobile supplementaire au-dela du web. L'equipe peut se concentrer sur le metier.
- **Effort minimal** : un seul code, un seul deploiement, un seul pipeline CI.
- **Mises a jour instantanees** : push une nouvelle version, les utilisateurs ont la mise a jour au prochain refresh (apres invalidation du Service Worker). Critique pendant la phase MVP ou on va beaucoup iterer.
- **Hors-ligne couvert** : Service Worker + IndexedDB (Dexie.js) gerent tout le niveau B (voir ADR-0003).
- **Acces materiel suffisant en V1** : scanners code-barres = HID (clavier), imprimantes thermiques = ESC/POS via reseau local ou WebUSB. On a tout ce qu'il faut sans natif.

## Consequences

### Positives

- Couverture Windows + macOS + Linux + Android + iOS avec un seul effort
- Cycle de deploiement court (commit → prod en quelques minutes)
- Pas de comptes payants App Store / Play Store
- Coherence parfaite entre desktop et mobile (meme UI)

### Negatives

- **iOS Safari limite** : pas de Web Bluetooth, notifications push restrictives. Mitigation : si un client iOS veut un terminal Bluetooth, on bascule sur Capacitor.
- **Perception** : certains commercants demandent "une app dans le store". Mitigation : argumentaire commercial qui explique les benefices PWA (mises a jour instantanees, pas de mises a jour a installer manuellement).
- **Decouverte d'installation** : l'utilisateur doit cliquer "Ajouter a l'ecran d'accueil". Mitigation : onboarding qui montre comment installer.
- **Acces hardware** : pour les TPE / TPV avances, on devra peut-etre passer en natif. Mitigation : architecture front-end agnostique du materiel via une couche d'abstraction.

### Mitigation principale

Si jamais on a besoin de natif en V2, **le code React ne change pas** : on ajoute Capacitor par-dessus, on republie dans les stores. Aucun travail jete.

## Liens

- ADR-0001 : Stack technique
- ADR-0003 : Offline niveau B
- [PWA support sur iOS (caniuse)](https://caniuse.com/?search=PWA)
- [Capacitor docs](https://capacitorjs.com/)

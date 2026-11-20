# TP 7 – Notes de sécurité sur les actions tierces

## Observations sur actions/setup-node

Relevé le 20/11/2026 sur [github.com/actions/setup-node](https://github.com/actions/setup-node) :

- **Créateur vérifié :** l'action est publiée par l'organisation `actions` (GitHub Actions), qui porte le badge de créateur vérifié sur le GitHub Marketplace.
- **Étoiles :** environ 5 000 (5 028 au moment du relevé).
- **Code source :** le dossier `src/`, écrit en TypeScript. Le code réellement exécuté par le runner est la version compilée dans `dist/`, comme l'indique `action.yml` (`main: dist/setup/index.js`).

## Test : actions/checkout@main au lieu de @v4

Avec `@v4`, le run affiche un avertissement indiquant que `actions/checkout@v4` cible Node.js 20. Avec `@main`, cet avertissement disparaît pour `checkout`, car la branche `main` de l'action cible déjà Node.js 24 (`using: node24`).

Le même fichier de workflow exécute donc un code différent selon la référence choisie : `@main` suit les derniers changements de l'action, sans que le workflow ait été modifié.

## Q3. Pourquoi faut-il épingler une version (@v4) plutôt que @main ?

`@main` désigne une branche, qui change à chaque nouveau commit de l'éditeur de l'action. Le workflow exécute alors du code qui n'a jamais été vérifié, et qui peut changer d'un run à l'autre :

- **Stabilité :** une modification incompatible sur `main` peut casser le workflow, sans aucun changement de notre côté.
- **Sécurité :** si le dépôt de l'action est compromis, le code malveillant poussé sur `main` est exécuté immédiatement, avec accès au code et aux secrets du workflow.
- **Reproductibilité :** deux runs du même commit peuvent utiliser deux versions différentes de l'action.

`@v4` limite les changements à des versions publiées et compatibles. Un tag reste cependant déplaçable par l'éditeur : la protection la plus forte consiste à épingler le SHA complet du commit, par exemple `actions/checkout@<sha de 40 caractères>`.

## Q4. Le badge « créateur vérifié » garantit-il qu'une action est sûre ? Justifiez.

Non. Le badge garantit seulement **l'identité de l'éditeur** : GitHub a vérifié que l'organisation qui publie l'action est bien celle qu'elle prétend être, par exemple via son nom de domaine. Il ne garantit pas que le code a été audité.

- Le compte ou le dépôt d'un éditeur vérifié peut être compromis, et un tag peut être redirigé vers un commit malveillant.
- L'action peut contenir une faille de sécurité ou dépendre de bibliothèques vulnérables.
- Une nouvelle version peut changer de comportement, par exemple envoyer des données vers l'extérieur.

Le badge est un indice de confiance, pas une preuve de sécurité.

## Q5. Citez deux vérifications à faire avant d'utiliser, en production, une action d'un fournisseur inconnu.

1. **Lire le code de l'action :** le fichier `action.yml` (ce qu'elle exécute, avec quels inputs), le code source et le code compilé réellement lancé (`dist/`). Il faut vérifier qu'elle ne fait rien d'inattendu, comme envoyer des secrets ou télécharger des scripts depuis Internet.
2. **Évaluer la fiabilité du dépôt :** activité et maintenance (commits récents, issues traitées), nombre d'utilisateurs, présence d'une politique de sécurité (`SECURITY.md`), et historique de failles signalées.

Une fois l'action adoptée, il faut aussi l'épingler par SHA complet et limiter les permissions du `GITHUB_TOKEN` (`permissions:`) au strict nécessaire.

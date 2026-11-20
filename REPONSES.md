# TP 6 – Variables d'environnement, Inputs et Outputs

## Partie 1 – Variables d'environnement avec GITHUB_ENV

### Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?

Chaque step s'exécute dans un nouveau processus shell. `echo "APP_NAME=my-app" >> $GITHUB_ENV` n'exporte pas la variable dans le shell en cours : la ligne est seulement écrite dans un fichier. Le runner lit ce fichier à la fin de la step, puis injecte les variables dans l'environnement des steps suivantes du job.

Dans la step « Set environment variables », la ligne `Same step: APP_NAME=''` le confirme : la variable est vide dans la step qui l'a définie. Elle est ensuite affichée par « Print environment variables ».

### Q2. Que constatez-vous dans le job verify-env ? Quelle est la portée de GITHUB_ENV ?

`$APP_NAME` est vide dans le job `verify-env` : la step affiche `APP_NAME in another job: ''`.

La portée de `GITHUB_ENV` est donc **le job** : une variable écrite dans `GITHUB_ENV` est disponible dans les steps suivantes du même job, mais pas dans les autres jobs. Chaque job tourne sur sa propre machine virtuelle, avec son propre environnement. Pour transmettre une valeur à un autre job, il faut passer par les outputs (`GITHUB_OUTPUT`) et `needs`.

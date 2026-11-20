# TP 7 – Actions tierces et actions personnalisées

## Partie 1 – Utiliser des actions tierces (uses et with)

### Q1. Pourquoi actions/checkout n'a-t-il pas besoin de with alors que setup-node en a besoin ?

Tous les paramètres de `actions/checkout` sont facultatifs et ont une valeur par défaut adaptée : par défaut, il récupère le dépôt du workflow, au commit qui a déclenché le run. Sans `with`, il fait donc déjà ce qu'on attend de lui.

`actions/setup-node` doit savoir quelle version de Node.js installer, et il n'a pas de version par défaut. Sans `node-version`, il n'installe rien et laisse la version déjà présente sur le runner, qui peut changer d'une image à l'autre. `with: node-version: 20` fixe la version utilisée et rend le build reproductible.

### Q2. Pourquoi le job verify ne retrouve-t-il pas report.txt sans l'artefact, alors que les deux jobs partagent le même dépôt ?

Chaque job tourne sur sa propre machine virtuelle, neuve et vide. `report.txt` est créé pendant le job `build`, sur sa machine, et disparaît quand ce job se termine.

Partager le même dépôt ne suffit pas : `actions/checkout` ne récupère que les fichiers commités, et `report.txt` n'est jamais commité. Le job `verify` ne fait d'ailleurs pas de checkout.

L'artefact sert de stockage intermédiaire : `upload-artifact` envoie le fichier sur GitHub à la fin de `build`, et `download-artifact` le récupère dans `verify`. `needs: build` garantit que l'artefact existe avant le téléchargement.

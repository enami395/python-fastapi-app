# TP 8 – Stratégies GitHub Actions

## Partie 1 – La matrice de base

### Q1. Combien de jobs sont générés ? Donnez la liste des combinaisons.

La matrice génère **6 jobs** : chaque valeur de `os` (2 valeurs) est combinée avec chaque valeur de `python-version` (3 valeurs), soit 2 × 3 = 6.

| # | os | python-version |
| --- | --- | --- |
| 1 | ubuntu-latest | 3.11 |
| 2 | ubuntu-latest | 3.12 |
| 3 | ubuntu-latest | 3.13 |
| 4 | windows-latest | 3.11 |
| 5 | windows-latest | 3.12 |
| 6 | windows-latest | 3.13 |

Le même job `test` est écrit une seule fois : GitHub le duplique pour chaque combinaison, en remplaçant `${{ matrix.os }}` et `${{ matrix.python-version }}` par les valeurs correspondantes.

## Partie 2 – exclude et include

### Q2. Combien de jobs au total ?

**6 jobs.** Le calcul se fait en trois temps :

1. **Matrice de base :** 2 × 3 = 6 combinaisons.
2. **`exclude` (appliqué en premier) :** `windows-latest` + `3.11` est retiré, il reste 5 combinaisons.
3. **`include` (appliqué ensuite) :** `ubuntu-latest` + `3.14` ne correspond à aucune combinaison existante, car `3.14` n'est pas dans la liste `python-version`. Il crée donc un nouveau job, ce qui donne 6 jobs.

| # | os | python-version | experimental |
| --- | --- | --- | --- |
| 1 | ubuntu-latest | 3.11 | |
| 2 | ubuntu-latest | 3.12 | |
| 3 | ubuntu-latest | 3.13 | |
| 4 | windows-latest | 3.12 | |
| 5 | windows-latest | 3.13 | |
| 6 | ubuntu-latest | 3.14 | true |

Grâce à `continue-on-error: ${{ matrix.experimental == true }}`, un échec du job 3.14 ne fait pas échouer le workflow. Pour les 5 autres jobs, l'expression vaut `false` : un échec reste bloquant.

### Q3. Que se passerait-il si vous tentiez d'exclure la combinaison 3.14 ? Pourquoi ?

L'exclusion n'aurait **aucun effet** : le job 3.14 serait quand même créé.

`exclude` s'applique avant `include`, et seulement sur la matrice de base, construite à partir des listes `os` et `python-version`. Or `3.14` n'est pas dans la liste `python-version` : la combinaison `ubuntu-latest` + `3.14` n'existe pas encore au moment où `exclude` est appliqué, il n'y a donc rien à retirer. `include` l'ajoute ensuite.

## Partie 3 – fail-fast

Échec contrôlé ajouté temporairement dans `tests/test_break.py` :

```python
import sys


def test_break():
    assert sys.platform != "win32"   # échoue uniquement sur Windows
```

### Q4. Quelle est la différence observée ? Dans quel cas choisiriez-vous true ? false ?

- **`fail-fast: true` (par défaut) :** dès qu'un job de la matrice échoue, GitHub annule tous les jobs encore en cours ou en attente. Les jobs déjà terminés gardent leur résultat. On obtient vite un signal d'échec, mais on ne sait pas si les combinaisons annulées auraient réussi.
- **`fail-fast: false` :** un échec n'interrompt pas les autres jobs. Tous vont jusqu'au bout, et on obtient le résultat complet de chaque combinaison.

**Quand choisir `true` :** quand un seul échec suffit à bloquer la livraison et qu'on veut économiser du temps et des minutes de runner, par exemple sur une pull request ou un pipeline de déploiement.

**Quand choisir `false` :** quand on veut savoir précisément quelles combinaisons échouent, par exemple pour diagnostiquer un problème propre à un système ou à une version de Python. C'est le cas ici : seul Windows échoue, et seul `fail-fast: false` le montre clairement.

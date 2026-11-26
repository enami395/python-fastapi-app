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

# Optimisation de la Production des Fruits Rouges en Serres

Projet du cours d'Optimisation (S6) — par Anasse Essalih. Le problème consiste à choisir, pour chaque serre disponible, le **scénario de production** (fruit, variété, type de plantation) et la **semaine de plantation** qui maximisent la marge totale sur l'ensemble de l'exploitation, sous contraintes sectorielles et de capacité.

## Contexte

- **Fruits étudiés** : Framboise et Mûre, chacun avec plusieurs variétés et types de plantation.
- **15 scénarios de production** distincts (combinaison fruit / variété / type de plantation / mois de plantation).
- **24 serres**, répartis en secteurs, avec des surfaces (SAU) différentes.
- **Contraintes sectorielles** :
  - Scénarios 4 et 5 : réservés au secteur 6.
  - Scénarios 12, 13, 14, 18 : réservés au secteur 5, et seulement pour les serres de moins de 2.87 ha.
- **Bonus** : possibilité de plafonner la production totale à un seuil ajustable.

Voir `projet.pdf` pour l'énoncé complet et `Optimisation.pdf` / `Optimisation.pptx` pour la présentation.

## Modélisation

Variables de décision binaires `X[i,j,k]` = 1 si la serre `i` est affectée au scénario `j` à la semaine de plantation `k`.

**Fonction objectif** (maximiser la marge) :

```
max  Σ X(i,j,t) · (CA(i,j,t) − CV(i,j))
```

avec :
- `CA(i,j,t')` : chiffre d'affaires — somme sur les semaines de production de `prod(j,t) × surface(i) × prix(j, t' + t + délai(j))`
- `CV(i,j)` : charge variable — `surface(i) × Charges(j)`

**Contraintes principales** :
- Chaque serre `i` est affectée à exactement un (scénario, semaine).
- Les scénarios 4/5 sont forcés à zéro hors secteur 6 ; les scénarios 12/13/14/18 sont forcés à zéro hors secteur 5 ou si la surface dépasse 2.87 ha.

C'est un **programme linéaire en variables binaires (MILP)**, résolu avec [PuLP](https://coin-or.github.io/pulp/) (solveur CBC par défaut).

## Données

| Fichier | Contenu |
|---|---|
| `Production.csv` | Profil de production hebdomadaire (W1…W37) par scénario : culture, variété, type de plantation, mois, délai avant production, durée |
| `Prices.csv` | Prix de vente par fruit, sur 90 semaines |
| `Charges_var.csv` | Coûts variables, main d'œuvre et niveau de risque par scénario |
| `Simulation.csv` | Liste des serres avec secteur et surface (SAU en ha) |

## Notebooks

- **`Script_1_Avril.ipynb`** — première version du modèle (grille de semaines `K=34`, indexation des prix démarrant en avril, sans correction de bord pour le dépassement de l'horizon de prix).
- **`Script_2_Janvier.ipynb`** — version aboutie : `K=35` semaines, calcul de `get_week(j)` (semaines calendaires valides par scénario), gestion explicite du dépassement d'index des prix (au-delà de 90 semaines), et contrainte bonus de plafond de production commentée/activable (`n`). C'est le script de référence pour les résultats finaux.
- `Untitled2.ipynb`, `Untitled3-Copy2.ipynb`, `Untitled.ipynb` — brouillons/itérations intermédiaires du même modèle (variantes de `K`, sans le filtrage par `get_week`).

## Stack

Python : `pulp` (MILP), `pandas` / `numpy` (données), `matplotlib` (visualisation), `datetime` (calcul des semaines calendaires).

## Pour exécuter

```bash
pip install pulp pandas numpy matplotlib
jupyter notebook Script_2_Janvier.ipynb
```

Le notebook charge les CSV du dossier courant, construit les tenseurs de marge `C_A`/`C_V`/`B`, assemble le problème PuLP, le résout (`prob.solve()`), puis affiche l'affectation optimale (serre, scénario, semaine) pour chaque serre.

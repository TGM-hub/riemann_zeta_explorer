# Riemann Zeta Explorer

Une exploration numérique et empirique de la fonction zêta de Riemann — l'un des problèmes non résolus les plus célèbres des mathématiques.

Ce repo ne prétend pas prouver quoi que ce soit. Il documente une démarche : partir des bases, comprendre les connexions inattendues, tenter d'appliquer le ML, échouer honnêtement, et comprendre pourquoi.

---

## L'hypothèse de Riemann

Posée en 1859 par Bernhard Riemann, elle affirme que tous les zéros non triviaux de la fonction

$$\zeta(s) = \sum_{n=1}^{\infty} \frac{1}{n^s}$$

ont leur partie réelle égale à ½. Autrement dit, ils se trouvent tous sur la **ligne critique** Re(s) = ½.

Personne n'a prouvé — ni réfuté — cette conjecture en 165 ans. C'est l'un des sept problèmes du millénaire, avec une récompense d'un million de dollars.

Ce qui rend le problème fascinant : les zéros non triviaux de ζ contrôlent la distribution des nombres premiers, et leur espacement suit la même loi statistique que les valeurs propres de matrices aléatoires en physique quantique. Deux connexions que personne n'a vraiment expliquées.

---

## Structure du repo

```
notebooks/
├── 01_series_definition.ipynb       Définition de ζ(s) comme série, convergence, problème de Bâle
├── 02_complex_plane.ipynb           ζ(s) dans le plan complexe, trajectoires, zéros
├── 03_primes_link.ipynb             Produit d'Euler, théorème des nombres premiers, Li(x)
├── 04_visualize_zeta.ipynb          Trajectoire de ζ(½+it), animation Manim
├── 05_zeros_stats.ipynb             Parsing des fichiers .dat, distribution GUE, densité
├── 06_ml_zeros.ipynb                Baseline, Ridge, LSTM, RealNVP — premiers résultats ML
├── 07_gram_points.ipynb             Fonction de Riemann-Siegel, points de Gram, déviations
└── 08_gram_features_ml.ipynb        Pipeline hybride, meilleur résultat, bilan

assets/
└── zeta_trajectory.gif              Animation Manim de ζ(½+it)

data/
└── zeros_*.dat                      Fichiers de zéros (format Booker/Hathi, non inclus)

zeta_anim.py                         Script Manim pour l'animation
```

---

## Ce qu'on explore

**Notebooks 01-04 — Comprendre ζ**

On part de la définition élémentaire — une somme d'inverses de puissances — et on remonte jusqu'à la visualisation de ζ dans le plan complexe. Le notebook 03 établit le lien avec les nombres premiers via le produit d'Euler et la formule de Riemann. Le notebook 04 montre la trajectoire de ζ(½+it) qui se densifie avec t, et les passages par l'origine qui sont les zéros non triviaux.

**Notebook 05 — Les données réelles**

On parse 9 millions de zéros non triviaux depuis les fichiers `.dat` de Booker & Hathi (2023), un format binaire sur 104 bits par zéro. On découvre que les gaps entre zéros consécutifs suivent la **loi GUE** (Gaussian Unitary Ensemble) — la même distribution que les espacements entre valeurs propres de matrices aléatoires en physique nucléaire. La densité des zéros suit exactement la formule théorique de Riemann ln(t/2π)/2π, vérifiée empiriquement à 4 décimales.

**Notebooks 06-08 — Tentatives ML**

L'objectif : prédire le prochain zéro non trivial. La difficulté fondamentale : les gaps ont un coefficient de variation de 0.42 — la variance est irréductible, conséquence directe de la loi GUE.

| Modèle | Feature | Amélioration vs baseline |
|---|---|---|
| Ridge | Gaps bruts | +8% |
| LSTM | Gaps bruts | −10% |
| Ridge | Déviations (ρ_n − g_n) | +21% |
| LSTM | Déviations | +20% |
| Ridge | Z(g_n) seules | −1% |
| Ridge | Combiné (déviations + Z) | +25% |
| **Pipeline hybride** | **Signe Z + combiné** | **+33%** |

Le meilleur résultat vient d'un pipeline en deux étapes : classifier l'intervalle de Gram comme bon ou mauvais via le **signe de Z(t)** (99.9% de précision, sans ML), puis prédire la déviation avec Ridge uniquement sur les bons intervalles.

**Résultat final** : 78.9% des zéros localisés à ±0.5, MAE = 27.7% de l'espacement moyen entre zéros.

C'est honnêtement loin d'une prédiction précise. La difficulté du problème — reflet de la profondeur de l'hypothèse de Riemann elle-même — résiste aux approches ML standard.

---

## Données

Les fichiers `.dat` proviennent du travail de Booker & Hathi (2023) — environ 3×10¹⁰ zéros calculés à une précision de 30 décimales. Le format est binaire propriétaire (104 bits par zéro, différences cumulées scalées par 2⁻¹⁰¹). Le parser est documenté dans le notebook 05.

Les fichiers ne sont pas inclus dans ce repo en raison de leur taille. Ils sont disponibles sur le site d'Andrew Odlyzko.

---

## Stack

```
Python 3.10+
numpy · scipy · mpmath
plotly · matplotlib
scikit-learn
pytorch
manim
```

---

## Références

- Riemann, B. (1859). *Über die Anzahl der Primzahlen unter einer gegebenen Grösse*
- Montgomery, H. (1973). *The pair correlation of zeros of the zeta function*
- Odlyzko, A. Tables of zeros of the Riemann zeta function — [odlyzko.com](https://odlyzko.com/zeta_tables/)
- Booker, A. & Hathi, S. (2023). *Rigorous computation of zeros of the Riemann zeta function*
- Shanker, O. *Neural Network prediction of Riemann zeta zeros* — [sites.google.com/site/riemannzetazeros](https://sites.google.com/site/riemannzetazeros)

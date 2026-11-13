# ADR-008 - Calculer la tournée de livraison de Noël avec le moins de distance possible.

## 1. Contexte

Ticket T-0531 : Les tournées calculées font 40 % de kilomètres de plus
que celles que les livreurs tracent à la main.

60 tournées de vélo cargo à 40 arrêts (jeu moyenne).

La contrainte qui compte est la distance totale parcourue, pour arriver à un meilleur résultat que les 40 % de plus que le tracé manuel. Le temps de calcul n'est pas critique, mais il ne doit pas dépasser 10 secondes pour 40 arrêts.

## 2. Options comparées

Deux au moins, la version naïve du legacy comprise. Une ligne par option, avec la complexité annoncée.

- Option A voisin : Complexité annoncée : O(n²).
- Option B deuxopt : Complexité annoncée : O(n²).
- Option C recuit : Complexité annoncée : O(nIter * CoutVoisin)

## 3. Mesures

Le protocole en cinq lignes, puis le tableau, puis la courbe.

- Machine : Intel Core i7-8650U @ 1,90 GHz, 15 Go de mémoire, Linux 7.0.0-38-generic, Node v22.19.0.
- Jeux : data/jeux/moyenne/colis.csv, livreurs.csv et hubs.csv, taille moyenne.
- Répétitions et échauffement : 3 répétitions, 1 échauffement.
- Statistique retenue : la médiane.
- Ce qui est mesuré : 
  - Temps de calcul pour 1, 5 et 20 tournées.
  - Mémoire utilisée (pic).
  - Qualité des tournées pour 60 tournées de 40 arrêts et pour 60 tournées de 9 arrêts (optimum par force brute).

| Option | 1 tournée | 5 tournées | 20 tournées | Mémoire (pic) |
| --- | --- | --- | --- | --- |
| A voisin | 0,14 ms | 0,64 ms | 0,40 ms | 0,02 Mo |
| B deuxopt | 1,34 ms | 0,85 ms | 2,60 ms | 0,04 Mo |
| C recuit | 10,08 ms | 19,52 ms | 63,67 ms | 3,0 Mo |

Qualité des tournées (`npm run noel` et `npm run noel -- --arrets 9`) :

| Option | km totaux, 60 tournées | Écart au meilleur | Écart à l'optimum, 9 arrêts | Optimum trouvé (9 arrêts) |
| --- | --- | --- | --- | --- |
| Legacy (ordre d'affectation) | 16 267,5 | +233,8 % | +67,2 % | 0 / 60 |
| A voisin | 5 706,7 | +17,1 % | +10,2 % | 3 / 60 |
| B deuxopt | 4 966,6 | +1,9 % | +0,4 % | 47 / 60 |
| C recuit | 4 873,8 | meilleur | +0,0 % | 60 / 60 |

Mesures : `bench/resultats/2026-11-13T15-08-06-noel.csv` (60 tournées de 40 arrêts) et `bench/resultats/2026-11-13T15-08-24-noel.csv` (60 tournées de 9 arrêts, optimum par force brute).


## 4. Décision

L'option de recuit est retenue pour calculer les tournées de livraison de Noël, car elle offre la meilleure qualité de tournées avec un écart minimal par rapport à l'optimum, tout en restant dans des temps de calcul acceptables.

## 5. Conséquences et limites

La décision entraîne une augmentation de la complexité du code et de la mémoire utilisée, mais elle permet d'obtenir des tournées plus optimisées. Cependant, ces coûts sont compensés par les gains significatifs en termes de distance parcourue et d'efficacité des tournées.

Domaine de validité : La décision est valable pour des tournées de livraison avec un nombre d'arrêts allant jusqu'à 40 et un volume de colis moyen. Au-delà de 100 arrêts, la performance pourrait se dégrader.

## 6. En trois phrases pour la directrice des opérations

Une nouvelle méthode de calcul des tournées de livraison de Noël a été mise en place, cette approche permet de réduire la distance totale parcourue par les livreurs. Les tests ont montré que cette méthode est plus efficace que les tournées tracées à la main, et presque aussi courte que le meilleur trajet possible. Ainsi, nous pouvons garantir des livraisons plus rapides et plus écologiques pour la période de Noël.
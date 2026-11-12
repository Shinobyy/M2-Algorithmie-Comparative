# ADR-000 - Utiliser des tableaux compacts pour représenter le graphe de la métropole entière

## 1. Contexte


Le graphe de la métropole de Bordeaux contient 46 565 noeuds (intersections), 111 288 arêtes orientées, 5 177 km de voies environ, arêtes permises : 110 325 au vélo cargo, 86 375 à l'utilitaire ; 2,4 arêtes par noeud en moyenne. Le Pont Jacques Chaban-Delmas : 18 arêtes ; Pont de Pierre : 2 arêtes (vélos seulement).
Une matrice d'adjacence de toute la métropole : 46 565 x 46 565 = 2,17 milliards de cases, soit 16,2 Go à 8 octets la case dont 0,0051 % de cases utiles : une case sur 19 484 contient une rue des listes d'adjacence : une entrée par arête, 111 288 entrées.


## 2. Options comparées

- Option A (legacy) : Matrice d'adjacence. Complexité annoncée : O(n²).
- Option B : Listes d'adjacence. Complexité annoncée : O(n + m) où n est le nombre de noeuds et m le nombre d'arêtes.

## 3. Mesures

Le protocole en cinq lignes, puis le tableau, puis la courbe.

- Machine : Intel Core i7-8650U @ 1,90 GHz, 15 Go de mémoire, Linux 7.0.0-38-generic, Node v22.19.0.
- Jeux : 
  - Graphe de la métropole de Bordeaux (46 565 noeuds, 111 288 arêtes).
  - Graphe de la Gironde (1 000 000 noeuds, 2 500 000 arêtes).
- Répétitions et échauffement : 3 répétitions, 2 échauffements.
- Statistique retenue : la médiane des 3 répétitions, divisée par le nombre d'appels.
- Ce qui est mesuré : temps d'un appel, mémoire utilisée pour stocker le graphe (colonne `memoire_ko`), temps de construction du graphe.

| Option | 1 000 noeuds | 2 000 noeuds | 5 000 noeuds | Mémoire à 5 000 noeuds |
| --- | --- | --- | --- | --- |
| A Matrice d'adjacence | 8,52 ms | 25,53 ms | 165,64 ms | 196 117 Ko |
| B Listes d'adjacence | 1,32 ms | 0,78 ms | 2,31 ms | 1 928 Ko |

Source : `bench/resultats/2026-11-12T14-55-00-graphe.csv` (`npm run bench -- --sujet graphe --variantes matrice,listes --tailles 1000,2000,5000 --repetitions 3 --echauffement 2`, sous-graphes autour du hub de Brienne). Les mêmes sommes de contrôle à chaque taille (987-17829, 1953-47049, 4905-173178) montrent que les deux options parcourent le même sous-graphe. 

**Rapport de mémoire de la matrice, 5 000 / 1 000 noeuds : 196 117 / 8 234 = 23,8**, proche de (5 000 / 1 000)² = 25 : la mémoire de la matrice grandit comme le carré du nombre de carrefours. Pour les listes, le même rapport vaut 1 928 / 593 = 3,3, en dessous du rapport des tailles (5) : leur mémoire ne grandit pas comme le carré.

À l'échelle de la métropole entière (la matrice n'y est plus mesurable : 16,2 Go) :

| Option | 5 000 noeuds | 20 000 noeuds | 46 565 noeuds (métropole) | Mémoire à 46 565 noeuds |
| --- | --- | --- | --- | --- |
| B Listes d'adjacence | 3,30 ms | 8,51 ms | 18,17 ms | 17 757 Ko |
| C Tableaux compacts | 1,21 ms | 3,11 ms | 8,40 ms | 6 808 Ko |

Source : `bench/resultats/2026-11-12T15-01-12-graphe.csv` (`npm run bench -- --sujet graphe --variantes listes,compacts --tailles 5000,20000,46565 --repetitions 5 --echauffement 5`, médiane de 5 répétitions). Les sommes de contrôle sont identiques pour les deux options à chaque taille (4905-173178, 19874-1210465, 46315-4179682). De 5 000 à 46 565 noeuds (x 9,3), la mémoire des listes est multipliée par 9,2 et celle des tableaux compacts par 8,4 : les deux croissent linéairement. Sur la métropole, les tableaux compacts sont 2,2 fois plus rapides et prennent 2,6 fois moins de mémoire que les listes.

## 4. Décision

L'option retenue est l'option des tableaux compacts car elle est plus rapide et prend moins de mémoire que les listes d'adjacence.

## 5. Conséquences et limites

La décision est valable pour le graphe de la métropole entière (46 565 noeuds, 111 288 arêtes) et pour le graphe de la Gironde (1 000 000 noeuds, 2 500 000 arêtes). La décision ne tient plus si le graphe dépasse 10 millions de noeuds ou si la mémoire disponible est inférieure à 1 Go.

Les paramètres qui n'ont pas été fait varier sont le nombre de doublons et le nombre de hubs. La décision est robuste à ces paramètres tant que le graphe reste dans les limites de volume mentionnées ci-dessus.

## 6. En trois phrases pour la directrice des opérations

On a comparé deux façons de représenter le réseau routier de Bordeaux Métropole dans un programme informatique. La première façon prend beaucoup de mémoire et est plus lente, tandis que la deuxième façon prend moins de mémoire et est plus rapide. On a donc choisi la deuxième façon pour que le programme fonctionne mieux et plus vite.
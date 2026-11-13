# ADR-000 - Calculer l'itinéraire le plus court entre deux points

## 1. Contexte

Ticket T-0733: 23 tournées à recalculer et 11 minutes de recalcul, les tablettes sont figées pendant ce temps.

## 2. Options comparées

Deux au moins, la version naïve du legacy comprise. Une ligne par option, avec la complexité annoncée.

- Option A (legacy) : ce que fait le code actuel. Complexité annoncée : O(...).
<!-- https://www.geeksforgeeks.org/dsa/time-and-space-complexity-of-dijkstras-algorithm/ -->
- Option B Dijkstra avec tas : Utilisation d'un algorithme de Dijkstra avec un tas pour optimiser la recherche du chemin le plus court. Complexité annoncée : O((V + E) log V).
<!-- https://en.wikipedia.org/wiki/A*_search_algorithm -->
- Option C A* : Utilisation d'un algorithme A* pour trouver le chemin le plus court. Complexité annoncée : O(b^d), où b est le facteur de branchement et d est la profondeur de la solution.

## 3. Mesures

Le protocole en cinq lignes, puis le tableau, puis la courbe.

- Machine : Intel Core i7-8650U @ 1,90 GHz, 15 Go de mémoire, Linux 7.0.0-38-generic, Node v22.19.0.
- Jeux : fichier de chemins `noeuds.csv.gz et aretes.csv.gz`. tailles 1 et 2.
- Répétitions et échauffement : 3 répétitions, aucun échauffement.
- Statistique retenue : la moyenne des trois répétitions.
- Ce qui est mesuré : 
  - le temps de calcul pour 1, 5 et 23 tournées (en millisecondes),
  - la mémoire utilisée à 23 tournées (en Ko).

| Option | 1 tournée | 5 tournées | 23 tournées | Mémoire à 23 tournées |
| --- | --- | --- | --- | --- |
| A Dijkstra sans tas | 5 952 ms | non mesuré | 2 min 55 s (extrapolé, `npm run pont`) | moins de 1 Mo |
| B Dijkstra avec tas | 21,0 ms | 56,6 ms | 211,3 ms | 13 728 Ko |
| C A* | 19,8 ms | 75,6 ms | 364,3 ms | 15 172 Ko |

Courbe : `bench/resultats/2026-11-13T10-30-52-chemins.svg` (générée par `npx tsx bench/courbe.ts <csv>`).

## 4. Décision

L'option retenue est l'option B Dijkstra avec tas. Elle offre un compromis optimal entre temps de calcul et consommation mémoire, répondant aux exigences du ticket T-0733.
L'option C A* est plus lente pour 23 tournées et consomme plus de mémoire, tandis que l'option A Dijkstra sans tas est trop lente pour être viable.

L'option C pourrait être envisagée seulement pour des requêtes de type un point vers un seul point.

## 5. Conséquences et limites

La décision entraîne une augmentation de la consommation mémoire par rapport à l'option A, mais reste dans des limites acceptables pour le système.
Le code est plus complexe que l'option A, nécessitant une compréhension approfondie de l'algorithme de Dijkstra avec tas.
Aucune dépendance externe n'a été ajoutée, ce qui maintient la simplicité du projet.

Domaine de validité : les paramètres qui ont été variés sont le nombre de tournées (1, 5, 23) et la taille des fichiers de chemins (1 et 2).

## 6. En trois phrases pour la directrice des opérations

L'option retenue est celle qui calcule le chemin le plus court entre deux points de manière efficace, en utilisant une méthode optimisée.
Elle permet de recalculer rapidement les itinéraires nécessaires pour les tournées, réduisant ainsi le temps d'attente des tablettes.
Cette solution équilibre bien la rapidité et l'utilisation de la mémoire, garantissant un fonctionnement fluide du système.

# ADR-000 - Titre de la décision (une ligne, à l'infinitif : "Retrouver un colis par son numéro de suivi")

Une page. Six rubriques, dans cet ordre. Copiez ce fichier en `ADR-00N.md` et remplacez chaque bloc.

## 1. Contexte

Le ticket du jour, les volumes, la contrainte qui compte. Trois à cinq lignes.

Exemple : la page "Où est mon colis" cherche un numéro de suivi parmi les événements de scan du jour, 70 000 lignes en septembre 2026, 1 200 000 sur un mois glissant. Elle est appelée 40 000 fois par jour. La contrainte est le temps de réponse par requête, pas le temps de construction.

## 2. Options comparées

- Option A (legacy) : la file est retriée en entier à chaque arrivée et décalée à chaque prise. Complexité annoncée : O(n log n) par arrivée et O(n) par prise, soit O(n²) au moins sur la soirée.
- Option B (tas binaire) : le plus urgent reste au sommet, le rang d'arrivée départage les égalités complètes. Complexité annoncée : O(log n) par arrivée et par prise.
- Option C (file à seaux) : une file par priorité, servie dans l'ordre d'arrivée. Complexité annoncée : O(1). Elle suppose que les colis arrivent dans l'ordre de leur création et qu'une priorité n'a qu'une fin de créneau.

## 3. Mesures

Le protocole en cinq lignes, puis le tableau, puis la courbe.

- Machine : Intel Core i7-8650U, 8 cœurs, 15 Go, Linux 7.0.0-38, Node v22.19.0.
- Jeu : `data/jeux/grande`, soirées de 1 000, 9 000 et 15 000 colis.
- Répétitions : 5, échauffement : 2.
- Statistique retenue : la médiane.
- Mesuré : durée de la soirée complète (arrivées et prises) et pic mémoire, sur le processus de Quai.

| File | 1 000 | 9 000 | 15 000 | Mémoire à 15 000 |
| --- | --- | --- | --- | --- |
| A legacy | 40,7 ms | 3 019,9 ms | 8 393,5 ms | 10,8 Mo |
| B tas | 3,8 ms | 28,6 ms | 49,0 ms | 2,1 Mo |
| C seaux | 0,5 ms | 2,4 ms | 5,4 ms | 0,9 Mo |

Épreuve : quand les colis arrivent dans l'ordre du fichier et non dans celui de leur création, le tas garde la somme de contrôle du legacy, mais la file à seaux la perd des 1 000 colis

Mesures : bench/resultats/2026-11-10T11-12-56-priorite.csv et bench/resultats/2026-11-10T11-13-31-priorite-arrivees.csv

Courbe : bench/resultats/2026-11-10T11-12-56-priorite.svg.

## 4. Décision

L'option retenue est la C, la file à seaux, pour sa rapidité et sa faible consommation mémoire.

## 5. Conséquences et limites

Ce que la décision coûte (mémoire, code, dépendance), ce qui n'a pas été mesuré. Trois à cinq lignes.

La décision coûte un peu de mémoire pour les files, mais pas de code supplémentaire. Elle ne tient pas compte des arrivées hors ordre de création, ni des priorités qui se chevauchent. Elle commence à poser problème à partir de 15 000 colis par contre

Il a été testé sur des volumes de 1 000 à 15 000 colis, avec un seul livreur et un seul hub, sans doublons. La décision ne tient plus à partir de 15 000 colis.

## 6. En trois phrases pour la directrice des opérations

Le choix retenu est un ordonnancement par tas, qui est une meilleure structure de la donnée, qui permet de gérer plus rapidement les assignations des priorités pour les livraisons.
La nouvelle méthode fait le même travail qu'avant en une vingtième de seconde et sert toujours les pharmacies dans l'ordre voulu.
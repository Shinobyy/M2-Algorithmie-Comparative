# ADR-005 - Rechercher une adresse le plus vite possible dans le referentiel

## 1. Contexte

Ticket du jour : La recherche de "ste catherine 33000 bordeaux" dans le référentiel d'adresses donne un résultat mauvais, et le temps de réponse est trop long.
La contrainte est premièrement l'exactitude de la recherche, secondement la vitesse de recherche.
Le référentiel fait 229k lignes, 1,5 Mo, et est mis à jour tous les jours. Il est stocké dans un fichier CSV, et chargé en mémoire par le serveur Node.js.

## 2. Options comparées

Deux au moins, la version naïve du legacy comprise. Une ligne par option, avec la complexité annoncée.

- Option A (legacy) : Actuellement, le code utilise la distance de Levenshtein contre toutes les adresses du référentiel, à chaque saisie. Complexité annoncée : O(n * m) où n est le nombre d'adresses et m la longueur de la saisie.
- Option B : les trigrammes. Utilisation de l'index de trigrammes en mémoire pour réduire le nombre d'adresses à comparer. Complexité annoncée : O(k * m) où k est le nombre d'adresses candidates après filtrage par trigrammes et m la longueur de la saisie.
- Option C : PostgreSQL avec l'extension `pg_trgm`. Stockage du référentiel dans une base PostgreSQL et utilisation de l'index de trigrammes fourni par l'extension. Complexité annoncée : O(k * m) similaire à l'option B, mais avec la robustesse et les optimisations de PostgreSQL.

## 3. Mesures

- Machine : Intel Core i7-8650U @ 1,90 GHz, 15 Go de mémoire, Linux 7.0.0-38-generic, Node v22.19.0.
- Jeux : référentiel de la métropole, coût d'une saisie mesuré sur 10, 20 et 50 saisies pour les deux index.
- Répétitions et échauffement : 3 répétitions, 1 échauffement.
- Statistique retenue : la médiane des 3 répétitions, divisée par le nombre de saisies.
- Ce qui est mesuré : temps d'une saisie, saisies justes, taille de l'index (colonne `index_mo`, pas `memoire_ko`), temps de construction de l'index. PostgreSQL tourne ici dans PGlite (dans Node), pas sur le serveur d'Efrei.

| Option | Temps | Saisies justes sur 300 | Taille de l'index | Construction sur toutes les adresses |
| --- | --- | --- | --- | --- |
| A - legacy | 3 586 ms | 10/10 (mesurée sur 10 saisies seulement) | aucun index | aucune |
| B - index de trigrammes en mémoire | 15,9 ms (10 : 15,9 ; 20 : 17,6 ; 50 : 15,9) | 288/300 (96,0 %) | 50,8 Mo en mémoire | 2,0 s |
| C - PostgreSQL `pg_trgm` (PGlite) | 200,3 ms (10 : 200,3 ; 20 : 202,5 ; 50 : 192,3) | 288/300 (96,0 %) | 42,3 Mo sur le disque de PostgreSQL | 9,1 s |

Sources : `bench/resultats/2026-11-12T11-23-20-adresses.csv` (coût d'une saisie, B et C) ; `bench/resultats/2026-11-12T12-42-16-exactitude.csv` 

## 4. Décision

Une phrase : L'option retenue est l'option des trigrammes, car elle est plus rapide (15.9ms) et plus robuste que l'option legacy, bien que la construction de l'index prenne 2 secondes. L'option PostgreSQL est plus lente (200ms) et nécessite une base de données, ce qui n'est pas nécessaire pour notre cas d'utilisation.

## 5. Conséquences et limites

Ce que la décision coûte (mémoire, code, dépendance), ce qui n'a pas été mesuré. Trois à cinq lignes.

Ce choix des trigrammes en mémoire implique une consommation de mémoire supplémentaire (50,8 Mo) pour stocker l'index. Le code est plus complexe que l'option legacy, mais il est maintenable et documenté. La dépendance à PostgreSQL est évitée, ce qui simplifie le déploiement et la maintenance.
Cependant, cette solution peut ne pas être adaptée si le référentiel d'adresses devient beaucoup plus grand à l'avenir, car la mémoire nécessaire pour l'index pourrait devenir un problème.


Domaine de validité : Cette décision est valable pour un référentiel d'adresses jusqu'à environ 1 million d'entrées. Au-delà de ce volume, la consommation de mémoire pour l'index pourrait devenir problématique, et il serait nécessaire de reconsidérer l'utilisation d'une base de données comme PostgreSQL pour gérer efficacement les recherches.

## 6. En trois phrases pour la directrice des opérations

Nous avons choisi une méthode pour trouver rapidement les adresses dans notre liste. Cette méthode est beaucoup plus rapide que l'ancienne et donne de bons résultats. Elle utilise un peu plus de mémoire, mais cela ne pose pas de problème pour notre usage actuel.
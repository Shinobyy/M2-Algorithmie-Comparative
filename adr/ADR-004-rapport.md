# ADR-004 - Sortir les cent premiers clients du mois pour le rapport de 6h

## 1. Contexte

Le premier jour ouvré du mois, à 6h, un cron lance sur la VM du site le rapport des cent premiers clients par chiffre d'affaires. Le legacy (2020) charge toutes les lignes de facturation du mois, les trie par client pour les regrouper, puis trie tous les clients par total.

## 2. Options comparées

- Option A, legacy, tri : trier les n lignes par client, additionner, trier les m clients par total, garder les cent premiers. Complexité annoncée : O(n log n).
- Option B tas de taille K : additionner par client avec une Map en une passe, puis garder les K = 100 meilleurs dans un tas dont le sommet est le moins bon des K. Complexité annoncée : O(n + m log K).
- Option C quickselect : additionner par client avec une Map, placer le K-ième meilleur à sa place par un pivot pseudo-aléatoire à graine fixe, puis ne trier que les K premiers. Complexité annoncée : O(n + m) en moyenne, O(n + m²) au pire.

## 3. Mesures

- Machine : Intel Core i7-8650U, 8 cœurs, 15 Go, Linux 7.0.0-38, Node v22.19.0.
- Jeu : data/jeux/mois (octobre 2026, graine 2026), 100 000, 1 000 000 et 3 000 000 de lignes.
- Répétitions : 5, échauffement : 2.
- Statistique retenue : la médiane.
- Mesuré : durée du rapport sur les lignes déjà chargées (regroupement et classement) et mémoire ajoutée par le calcul (heapUsed relevé avant et après l'appel, lu comme un ordre de grandeur).

| Option | 100 000 | 1 000 000 | 3 000 000 | Mémoire à 3 000 000 |
| --- | --- | --- | --- | --- |
| A tri | 88,7 ms | 1 016,3 ms | 2 092,7 ms | 63 732 Ko (62 Mo) |
| B tas | 13,2 ms | 51,8 ms | 132,6 ms | 300 Ko |
| C quickselect | 6,7 ms | 50,5 ms | 146,1 ms | 234 Ko |

Les trois options rendent la même somme de contrôle à chaque taille, et les trois tests passent pour les trois

Mesures : bench/resultats/2026-11-10T15-10-31-rapport.csv

Courbe : bench/resultats/2026-11-10T15-10-31-rapport.svg

Grille : adr/GRILLE-ADR-004.md, A 38, B 45, C 40.

## 4. Décision

L'option retenue est la B, le tas de taille K : elle divise la mémoire du rapport par 200 et sa durée par 15 à trois millions de lignes, avec un pire cas garanti, en réutilisant le TasBinaire déjà écrit et testé

## 5. Conséquences et limites

Le code du rapport passe de deux tris à une Map et un tas : peu de code en plus, mais une idée de plus à connaître (le sommet est le moins bon des K).
Le quickselect est écarté parce qu'il est plus difficile à lire et que son pire cas n'est que rendu improbable.

Domaine de validité : mesuré de 100 000 à 3 000 000 de lignes, pour K = 100 et 1 900 clients, sur une seule machine. La mémoire mesurée est celle du calcul. les trois millions de lignes restent chargées en mémoire avant le rapport, quelle que soit l'option, et ce chargement n'a pas été mesuré ici. Si K devenait proche du nombre de clients, l'avantage du tas sur un tri des totaux disparaîtrait.

## 6. En trois phrases pour la directrice des opérations

Le rapport des cent premiers clients du mois sortira en un dixième de seconde au lieu de deux secondes.
Il demandera deux cents fois moins de place sur la machine du site, ce qui lui laisse de la marge quand les volumes du mois grandiront.
Le classement reste exactement le même qu'aujourd'hui, y compris quand deux clients ont le même total.

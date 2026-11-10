# ADR-001 - Affecter les colis aux livreurs dans le budget de nuit de décembre

## 1. Contexte

L'étape 2 de l'aiguillage affecte chaque colis à un livreur la nuit, avec un budget de 43 minutes.
En décembre il y avait 15 000 colis pour 85 livreurs. Chaque couple colis-livreur examiné coûte une requête à la base (3,26 ms la nuit). Ce qui compte, c'est donc le nombre de couples et pas le temps de calcul.

## 2. Options comparées

- Option A (legacy) : chaque colis examine les 85 livreurs, sans break. Complexité : O(n²).
- Option B (tri) : trie avant d'affecter mais examine toujours 85 livreurs par colis. Complexité : O(n log n).
- Option C (map) n'examine que les livreurs utiles, environ deux fois moins. Complexité : O(n).

## 3. Mesures

- Machine : i7-8650U, 15 Go, Ubuntu, Node v22.19.0.
- Jeux : 1 000 à 10 000 colis (moyenne), 100 000 pour le legacy (j1-chrono), couples de 9 000 à 20 000 (grande).
- Répétitions et échauffement : 5 répétitions, 200 tours.
- Statistique retenue : la médiane
- Ce qui est mesuré : le temps, la mémoire, et les couples colis-livreur (avec npm run couples). Même somme de contrôle pour les trois.

| Option | 1 000 | 10 000 | 100 000 | Mémoire |
| --- | --- | --- | --- | --- |
| A | 31,9 ms | 2 611,6 ms | 555 630 ms | 4,3 Mo |
| B | 5,8 ms | 37,9 ms | — | 2,3 Mo |
| C | 1,6 ms | 18,0 ms | — | 5,1 Mo |

B et C ne sont pas mesurées à 100 000 colis : le harness s'arrête à 10 000, la valeur du legacy vient de l'atelier 2

A 15 000 colis, A et B font 1 271 770 couples, soit environ 69 min. C en fait 651 031, soit environ 35 min

Fichiers bench/resultats/2026-11-09T14-44-31-affectation.csv et bench/resultats/2026-11-09T15-12-29-couples.csv
Courbe : bench/resultats/2026-11-09T14-44-31-affectation.svg.

## 4. Décision

On retient C, seule option a tenir les 43 minutes à 15 000 colis (35 min contre 69) : c'est le temps qui a tranché la dessus.

## 5. Conséquences et limites

C ajoute une nouvelle structure (une map) mais aucune dépendance. La mémoire ne départage pas les options. Tout est calculé avec le coût d'une requête la nuit (3,26 ms) : le matin, quand la base est partagée, une requête coûte 14,8 ms, et C prendrait 651 031 × 14,8 ms = 160 min, bien au-delà des 43.

Domaine de validité : on a fait varier le volume (9 000 à 20 000 colis), mais pas les 85 livreurs, les 3 hubs ni l'heure. C tient jusqu'à environ 16 600 colis par nuit (49 min à 18 000).

## 6. En trois phrases pour la directrice des opérations

Chaque nuit, il faut attribuer 15 000 colis à 85 livreurs en 43 minutes, et l'ancienne façon de faire demande à chacun des 85 livreurs pour chaque colis, ce qui prend 69 minutes. La nouvelle façon ne demande qu'à deux fois moins de livreurs : 35 minutes, la seule qui tient dans le temps. Elle tient jusqu'à environ 16 600 colis par nuit ; au-delà, il faudra revoir ce choix.
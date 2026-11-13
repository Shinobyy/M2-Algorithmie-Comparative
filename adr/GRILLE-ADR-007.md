# Grille comparative - ADR-007 (recalculer les tournées touchées quand le pont Chaban-Delmas se lève, ticket T-0733)

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids (somme = 10) |
| --- | --- | --- |
| Temps | durée du recalcul des 23 tournées touchées le 8 septembre, pont levé (jeu moyenne) | 4 |
| Mémoire | pic mémoire du recalcul des 23 tournées | 1 |
| Lisibilité | on part de 5, -1 par idée à connaître pour lire la variante | 2 |
| Robustesse | mêmes distances au mètre près que les autres options (sommes de contrôle), et ce qui pourrait les rendre fausses | 2 |
| Coût de mise en oeuvre | lignes à écrire, structure à tenir, données à préparer, dépendances | 1 |

Le temps pèse le plus : pendant le recalcul, les tablettes sont figées et les livreurs touchés roulent vers un pont fermé. La mémoire pèse peu : les trois options tiennent sous 20 Mo.

## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A Dijkstra sans tas | 2 | 5 | 4 | 5 | 4 | 35 |
| B Dijkstra avec tas | 5 | 4 | 3 | 5 | 4 | 44 |
| C A* | 5 | 4 | 2 | 4 | 3 | 39 |

Le legacy du 8 septembre (11 minutes) n'est pas noté : il est cité en rubrique 1 de l'ADR.

Barèmes

- Temps, 23 tournées : 5 sous 1 s, 4 sous 10 s, 3 sous 1 min, 2 sous 11 min (le 8 septembre), 1 au-delà.
- Mémoire, 23 tournées : 5 sous 10 Mo, 4 sous 100 Mo, 3 sous 1 Go, 2 sous 4 Go, 1 au-delà.
- Lisibilité : 5, moins un point par idée à connaître.
- Robustesse : 5 si les distances sont les mêmes que les autres options à toutes les tailles ; 4 si elles le sont mais dépendent d'une hypothèse qui peut casser en silence ; 2 si une partie des tournées ne peut pas être traitée ; 1 si les distances diffèrent.
- Coût : 5 si rien ne change ; 4 pour quelques lignes sans structure nouvelle, ou une structure déjà écrite et testée ; 3 pour une structure ou une donnée de plus à tenir ; 2 pour une donnée à préparer à chaque lancement ; 1 pour une dépendance ajoutée.

Mesures : `bench/resultats/2026-11-13T10-27-36-chemins.csv` (simple et tas, 1 et 2 tournées, 3 répétitions), `bench/resultats/2026-11-13T10-30-52-chemins.csv` (tas et astar, 1, 5 et 23 tournées, 5 répétitions, échauffement 3, médiane) et `npm run pont` (simple extrapolé à 23 tournées). Sommes de contrôle identiques entre options à chaque taille (5c36f493, ee2fe1d0, 6cc66ea5, c720f156).

- Temps : A met 5,95 s pour une tournée, soit 2 min 55 s pour les 23 (`npm run pont`), sous les 11 minutes du 8 septembre mais au-dessus de la minute, donc 2. B met 211 ms pour les 23, donc 5. C met 364 ms, donc 5.
- Mémoire : A ne garde que deux tableaux de 46 565 cases (moins de 1 Mo), 5. B prend 13 728 Ko (13,4 Mo), 4. C prend 15 172 Ko (14,8 Mo), 4.
- Lisibilité : A demande une idée, fixer le carrefour le plus proche et relâcher ses voisins, 4. B en ajoute une, le tas où un carrefour peut entrer plusieurs fois et dont seule la première sortie compte, 3. C en ajoute une troisième, l'heuristique qui ordonne le tas sans entrer dans les distances, avec une recherche par arrêt, 2.
- Robustesse : les trois rendent les mêmes distances à chaque taille. A et B ne supposent que des longueurs positives, 5. C suppose en plus que le vol d'oiseau ne dépasse jamais la route : si les rues étaient pondérées en durées et non plus en mètres, A* rendrait des distances fausses sans erreur, 4.
- Coût : A, une vingtaine de lignes sans structure, 4. B réutilise le tas de l'atelier 6, déjà écrit et testé, plus quatre lignes, 4. C demande en plus les coordonnées de chaque carrefour, le calcul du vol d'oiseau et la remise à zéro des cases touchées entre deux arrêts, 3. Aucune option n'ajoute de dépendance.

## 3. La sensibilité

Avec temps 5 et lisibilité 1 (mémoire 1, robustesse 2, coût 1) : A 33, B 46, C 42. B reste devant.

En gardant les autres poids, A vaut 2 × poids du temps + 27, B 5 × poids du temps + 24 et C 5 × poids du temps + 19 : B passe devant A dès que le temps pèse 2 (à 1, égalité à 29). B et C ont la même note de temps, donc C − B = −5 quel que soit le poids du temps : C ne passe jamais devant B sur ce critère. La décision n'est pas fragile ; elle ne changerait que si la note de temps de C dépassait celle de B, ce qui n'arrive pas tant que les deux restent sous la seconde.

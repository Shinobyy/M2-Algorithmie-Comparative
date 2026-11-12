# Grille comparative - ADR-006 (ranger le réseau routier de Bordeaux Métropole en mémoire, préalable au ticket T-0733)

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids |
| --- | --- | --- |
| Temps | durée d'un parcours du graphe de la métropole entière (46 565 carrefours) | 2 |
| Mémoire | mémoire de la représentation pour la métropole entière | 4 |
| Lisibilité | on part de 5, -1 par idée à connaître pour lire la variante | 2 |
| Robustesse | même résultat que les autres options (`npm test`, sommes de contrôle), et à toutes les tailles jusqu'à la métropole | 1 |
| Coût de mise en oeuvre | lignes à écrire, structure à tenir, données à préparer, dépendances | 1 |

La mémoire pèse le plus : c'est elle qui décide si la métropole entière tient dans le processus, et elle grossira avec Libourne et le Bassin en 2027.

## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A legacy, matrice d'adjacence | 1 | 1 | 5 | 2 | 5 | 23 |
| B listes d'adjacence | 4 | 4 | 4 | 5 | 4 | 41 |
| C tableaux compacts | 5 | 5 | 3 | 5 | 3 | 44 |

Barèmes

- Temps, métropole entière : 5 sous 10 ms, 4 sous 100 ms, 3 sous 1 s, 2 sous 10 s, 1 au-delà ou impossible.
- Mémoire, métropole entière : 5 sous 10 Mo, 4 sous 100 Mo, 3 sous 1 Go, 2 sous 4 Go, 1 au-delà.
- Lisibilité : 5, moins un point par idée à connaître.
- Robustesse : 5 si le parcours est le même que les autres options à toutes les tailles, métropole comprise ; 3 si le résultat est le même mais une partie des données est perdue ; 2 si la métropole ne peut pas être traitée ou des données sont perdues en silence ; 1 si `npm test` échoue.
- Coût : 5 si rien ne change ; 4 pour quelques lignes sans structure nouvelle ; 3 pour une structure à écrire, à tester et à reconstruire à chaque modification du graphe ; 2 pour une donnée à préparer à chaque lancement ; 1 pour une dépendance ajoutée.

Mesures : `bench/resultats/2026-11-12T14-55-00-graphe.csv` (matrice et listes, 1 000 à 5 000 carrefours, 3 répétitions, échauffement 2, médiane) et `bench/resultats/2026-11-12T15-01-12-graphe.csv` (listes et tableaux compacts, 5 000 à 46 565 carrefours, 5 répétitions, échauffement 5, médiane). Sommes de contrôle identiques entre options à chaque taille.

- Temps : A ne peut pas être lancée sur la métropole (Node s'arrête à court de mémoire), donc 1 ; à 5 000 carrefours elle met déjà 165,64 ms. B met 18,17 ms sur la métropole, donc 4. C met 8,40 ms, donc 5.
- Mémoire : A demanderait 16,2 Go pour la métropole (46 565 x 46 565 cases à 8 octets) ; sa mémoire est multipliée par 23,8 de 1 000 à 5 000 carrefours, comme le carré ; donc 1. B prend 17 757 Ko (17,3 Mo), donc 4. C prend 6 808 Ko (6,6 Mo), donc 5.
- Lisibilité : A se lit sans idée nouvelle (une case par couple de carrefours, lire la ligne de u), 5. B demande une idée, une liste de voisins par carrefour, 4. C en demande deux : le tableau `debut` des positions cumulées et le rangement des arêtes bout à bout (lignes 73 à 78 de `bench/variantes/representations.ts`), 3.
- Robustesse : le test « matrice, listes et tableaux compacts parcourent le même sous-graphe » passe. A ne traite pas la métropole et écrase en silence la seconde de deux arêtes parallèles (une rue et sa contre-allée), 2. B et C rendent le même parcours à toutes les tailles, métropole comprise (46315-4179682), 5.
- Coût : A est déjà la structure du legacy, 5. B, ce sont quelques lignes (le À COMPLÉTER de l'atelier, dix lignes), 4. C est une structure de plus à écrire et à tester, et à reconstruire entièrement à chaque arête modifiée (un pont levé), 3.

## 3. La sensibilité

Avec temps 5 et lisibilité 1 (mémoire 4, robustesse 1, coût 1) : A 21, B 49, C 56. C reste devant.

En gardant les autres poids, B vaut 4 × poids du temps + 33 et C 5 × poids du temps + 34 : C reste devant B quel que soit le poids du temps (C − B = poids du temps + 1), la décision ne bascule pas sur le temps. Elle bascule sur la mémoire et la lisibilité : C − B = poids de la mémoire − 1 avec les autres poids fixés, donc B ne passe devant que si la mémoire ne pèse presque plus (0) ; et C − B = 5 − poids de la lisibilité, donc B passe devant si la lisibilité pèse 6 ou plus. A ne passe devant dans aucun de ces cas.

L'écart entre C et B est de 3 points seulement : si l'on doit modifier souvent le graphe (ponts levés, rues fermées), le coût de C pèse plus lourd et l'écart se réduit encore.

# Grille comparative - ADR-003 (la file du quai, servir le plus urgent d'abord)

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids (somme = 10) |
| --- | --- | --- |
| Temps | durée de la soirée complète (arrivées et prises) à 15 000 colis, et croissance de 1 000 à 15 000 | 3 |
| Mémoire | pic mémoire relevé par le harness à 15 000 colis | 1 |
| Lisibilité | on part de 5, -1 par idée à connaître pour comprendre le code (tas, rang d'arrivée, hypothèse cachée) | 2 |
| Robustesse | même ordre de sortie que le legacy (somme de contrôle), y compris quand les colis arrivent dans le désordre ; file vide, priorité inconnue | 3 |
| Coût de mise en oeuvre | lignes changées, dépendances, structure nouvelle pour l'équipe | 1 |

La robustesse pèse autant que le temps : une file qui sert un colis standard avant une pharmacie est une erreur métier, pas une lenteur.

## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A (legacy) | 2 | 3 | 5 | 4 | 5 | 36 |
| B (tas) | 5 | 4 | 3 | 4 | 3 | 40 |
| C (seaux) | 5 | 5 | 4 | 1 | 4 | 35 |

Barème du temps (fixé avant de mesurer, sur la soirée à 15 000 colis) : 5 sous 100 ms, 4 sous 1 s, 3 sous 5 s, 2 sous 10 s, 1 au-delà.

Mesures : `bench/resultats/2026-11-10T11-12-56-priorite.csv` et `bench/resultats/2026-11-10T11-13-31-priorite-arrivees.csv`.

- Temps : A met 8 393,5 ms à 15 000 colis et grandit en n² (x 74 entre 1 000 et 9 000), donc 2 ; B met 49,0 ms, donc 5 ; C met 5,4 ms, donc 5.
- Mémoire : 10,8 Mo pour A, 2,1 Mo pour B, 0,9 Mo pour C. Les relevés sont stables d'une répétition à l'autre, l'écart est réel mais reste petit à cette échelle.
- Lisibilité : A se lit en une ligne (on retrie, on prend le premier) ; B demande de connaître le tas et de comprendre pourquoi une Map des rangs d'arrivée départage les égalités ; C est simple à lire mais repose sur une hypothèse qui n'est écrite que dans un commentaire.
- Robustesse : A est la référence et B garde sa somme de contrôle quand les colis arrivent dans le désordre (89c52d41 à 1 000 colis) ; C la perd dès 1 000 colis (bc776f41) car elle ignore la fin de créneau et l'heure de création, donc 1. Les trois rendent `undefined` sur une file vide ; une priorité inconnue n'est rejetée par aucune (A et B la trient au hasard, C plante sur `files[-1]`), d'où 4 et pas 5 pour A et B.
- Coût : A ne change rien (5). B s'appuie sur `tas.ts` (59 lignes, sans dépendance) et ajoute la Map des rangs (3). C tient en une quinzaine de lignes sans dépendance (4).

## 3. La sensibilité

Avec temps 5 et lisibilité 1 (mémoire 1, robustesse 3, coût 1) : A 35, B 47, C 41. B reste devant, et son avance sur A grandit.

En gardant les autres poids, A vaut 2 × poids du temps + 30 et B vaut 5 × poids du temps + 25 : la décision bascule vers A quand le temps pèse moins de 1,7. Avec un poids de 3, on a de la marge.

Le vrai point de bascule est entre B et C, et il porte sur la robustesse : B − C = 3 × poids de la robustesse − 4, donc C passe devant dès que la robustesse pèse 1. Choisir C reviendrait à garantir que les colis arrivent toujours sur le quai dans l'ordre de leur création, ce que l'épreuve des arrivées dans le désordre contredit.

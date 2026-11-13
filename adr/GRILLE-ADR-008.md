# Grille comparative - gabarit

La grille sert à décider, pas à classer par vitesse. Les poids dépendent du ticket : un traitement de nuit
qui tient sa fenêtre supporte d'être lent, un traitement qui en déborde ne le supporte plus, une page appelée
40 000 fois par jour ne l'a jamais supporté. Les poids et le barème de chaque note se fixent avant de mesurer.

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids (somme = 10) |
| --- | --- | --- |
| Temps | médiane du harness à 20 tournées de 40 arrêts (jeu moyenne) | 3 |
| Mémoire | pic du harness, matrice comprise | 1 |
| Lisibilité | on part de 5, -1 par idée à connaître pour dire ce que fait la fonction, ce qui se passe quand un échange n'améliore pas, où changer la graine | 2 |
| Robustesse | écart moyen à l'optimum (force brute) sur 60 tournées de neuf arrêts, `npm run noel -- --arrets 9` | 3 |
| Coût de mise en oeuvre | longueur de la fonction, réglages à tenir, dépendances | 1 |


## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A voisin | 5 | 5 | 4 | 2 | 4 | 38 |
| B deuxopt | 5 | 5 | 2 | 5 | 4 | 43 |
| C recuit | 5 | 5 | 1 | 5 | 3 | 40 |

Total pondéré = somme des notes multipliées par les poids. Une note de temps ne se donne pas sans une mesure dans `bench/resultats/` : le CSV du harness et celui de `npm run couples` quand la production paie des requêtes.

Barème de la note de temps, à écrire ici avant de mesurer : la durée qui vaut 5, celle qui vaut 1 et ce qui sépare les deux. Il se fixe sur le budget du ticket (la fenêtre, le temps de réponse attendu), pas sur la plus rapide des options.

## 3. La sensibilité

Poussez le poids du temps à 5 et celui de la lisibilité à 1 : la décision change-t-elle ? Puis cherchez le poids du temps à partir duquel elle bascule. Une phrase chacun. Si la décision bascule près des poids choisis, dites-le dans l'ADR : c'est une décision fragile, le CTO doit le savoir.

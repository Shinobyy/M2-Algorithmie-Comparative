# Grille comparative - ADR-004 (le rapport de 6h, les cent premiers clients du mois)

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids |
| --- | --- | --- |
| Temps | durée du rapport à trois millions de ligne | 2 |
| Mémoire | pic mémoire relevé à trois millions de lignes, lu comme un ordre de grandeur | 4 |
| Lisibilité | on part de 5, -1 par idée à connaître pour lire la variante | 2 |
| Robustesse | classement exact sur les trois tests de tests/rapport.test.ts, et pire cas garanti ou non | 1 |
| Coût de mise en oeuvre | lignes changées, structure à écrire, données à préparer, dépendances | 1 |

La mémoire pèse le plus : le rapport tourne à 6h sur la VM du site, et c'est elle qui grossit avec le mois.

## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A legacy, tri | 3 | 3 | 5 | 5 | 5 | 38 |
| B tas de taille K | 5 | 5 | 3 | 5 | 4 | 45 |
| C quickselect | 5 | 5 | 2 | 3 | 3 | 40 |

Barèmes

- Temps : 5 sous 0,5 s, 4 sous 2 s, 3 sous 10 s, 2 sous 60 s, 1 au-delà.
- Mémoire : 5 sous 10 Mo, 4 sous 50, 3 sous 200, 2 sous 1 Go, 1 au-delà ; deux relevés à moins d'un facteur 2 l'un de l'autre prennent la même note.
- Lisibilité : 5, moins un point par idée à connaître (le tas et son sommet qui est le moins bon ; le pivot, la descente dans une seule moitié et le tirage à graine fixe).
- Robustesse : 5 si le classement est exact sur les trois tests avec un pire cas garanti ; 3 si exact mais pire cas quadratique rendu improbable par un pivot tiré au hasard ; 2 si un cas courant donne un classement faux en silence ; 1 si npm test échoue.
- Coût : 5 si rien ne change ; 4 pour quelques lignes et une structure déjà écrite ou fournie ; 3 pour une structure à écrire et à tester ; 2 pour une donnée à préparer à chaque lancement ; 1 pour une dépendance ajoutée.

Mesure : bench/resultats/2026-11-10T15-10-31-rapport.csv (5 répétitions, échauffement 2, médiane, même somme de contrôle 1f2ec5a2 pour les trois à trois millions de lignes).

- Temps : A met 2 092,7 ms, donc 3 ; B met 132,6 ms et C 146,1 ms, donc 5. A est sur la frontière : ses cinq relevés vont de 1 999 à 2 222 ms, il prendrait 4 un jour sur cinq.
- Mémoire : A monte à 63 732 Ko (62 Mo), donc 3 ; B reste à 300 Ko et C à 234 Ko, à moins d'un facteur 2 l'un de l'autre, donc 5 tous les deux.
- Lisibilité : A se lit sans idée nouvelle (trier, regrouper, trier), 5 ; B demande le tas et son sommet qui est le moins bon des K, 3 ; C demande le pivot, la descente dans une seule moitié et le tirage à graine fixe, 2.
- Robustesse : les trois tests passent pour les trois options (même classement, égalités départagées par le numéro de client, moins de clients que K). A et B ont un pire cas garanti (O(n log n) et O(m log K)), 5 ; C a un pire cas quadratique, rendu improbable par le pivot pseudo-aléatoire, 3.
- Coût : A ne change rien, 5 ; B ajoute une dizaine de lignes sur le `TasBinaire` déjà écrit et testé à l'atelier 6, 4 ; C est une structure à maintenir et à tester en plus, 3.

## 3. La sensibilité

Avec temps 5 et lisibilité 1 (mémoire 4, robustesse 1, coût 1) : A 42, B 57, C 53. B reste devant.

En gardant les autres poids, A vaut 3 × poids du temps + 32, B vaut 5 × poids du temps + 35 et C 5 × poids du temps + 30 : B reste devant quel que soit le poids du temps, même à zéro, la décision ne bascule pas sur le temps. Elle bascule sur la mémoire : B − A = 2 × poids de la mémoire − 1, donc A ne repasse devant que si la mémoire ne compte plus du tout. C ne passe devant B que si la lisibilité, la robustesse et le coût pèsent ensemble zéro.

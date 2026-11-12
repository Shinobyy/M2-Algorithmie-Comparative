# Grille comparative - ADR-005 (rapprocher une adresse mal saisie du référentiel)

## 1. Les critères et leur poids pour ce ticket

| Critère | Ce qu'on mesure ou ce qu'on juge | Poids |
| --- | --- | --- |
| Temps | temps d'une saisie sur la Gironde (trois fois plus d'adresses que la métropole : temps mesuré sur la métropole x 3) | 3 |
| Mémoire | mémoire ajoutée au processus du site à l'échelle de la Gironde (taille de l'index x 3) | 1 |
| Lisibilité | on part de 5, -1 par idée à connaître pour dire ce que fait la fonction, ce qui se passe sans numéro, où changer le nombre de candidats | 1 |
| Robustesse | `npm test`, puis saisies justes sur 300, par sorte de saisie, et le ticket T-0488 lui-même | 3 |
| Coût de mise en oeuvre | lignes à écrire, dépendances en production, données à préparer et à entretenir | 2 |

Temps et robustesse pèsent le plus : le support attend la réponse pendant la saisie, et une adresse rapide mais fausse livre au mauvais endroit.

## 2. Les options, note de 1 à 5 par critère

| Option | Temps | Mémoire | Lisibilité | Robustesse | Coût | Total pondéré |
| --- | --- | --- | --- | --- | --- | --- |
| A legacy, force brute | 1 | 5 | 4 | 2 | 5 | 28 |
| B index de trigrammes en mémoire | 5 | 4 | 2 | 4 | 3 | 39 |
| C PostgreSQL pg_trgm | 4 | 5 | 2 | 4 | 2 | 35 |

Barèmes (fiche atelier 8, fixés avant de mesurer)

- Temps, une saisie sur la Gironde : 5 sous 0,1 s, 4 sous 1 s, 3 sous 5 s, 2 sous 10 s, 1 au-delà.
- Mémoire ajoutée au processus du site : 5 si rien, 4 sous 500 Mo, 3 sous 2 Go, 2 sous 8 Go, 1 au-delà.
- Lisibilité : 5, moins un point par idée à connaître.
- Robustesse : 5 si toutes les saisies sont retrouvées ; 4 si au moins 95 % et les échecs s'expliquent ; 3 entre 80 et 95 % ; 2 si la méthode se trompe sur T-0488 ou n'a été mesurée que sur dix saisies ; 1 si `npm test` échoue.
- Coût : 5 si rien ne change ; 4 pour quelques lignes sans donnée à préparer ; 3 pour quelques lignes et un index à reconstruire à chaque version du référentiel ; 2 pour une extension, une table et un index à tenir dans la base ; 1 pour une dépendance ajoutée en production.

Mesures : `bench/resultats/2026-11-12T11-23-20-adresses.csv` (B et C, 10, 20 et 50 saisies, 3 répétitions, échauffement 1, médiane) et `bench/resultats/2026-11-12T12-42-16-exactitude.csv` (300 saisies pour B et C, 10 pour A).

- Temps : A met 3 586 ms par saisie sur la métropole, soit environ 10,8 s sur la Gironde, donc 1. B met 15,9 ms (159,16 ms pour 10 saisies), soit environ 48 ms sur la Gironde, donc 5. C met 200,3 ms (2 002,68 ms pour 10 saisies), soit environ 600 ms, donc 4. C est mesuré dans PGlite, pas sur le PostgreSQL d'Efrei.
- Mémoire : A n'ajoute aucun index, 5. L'index de B fait 50,8 Mo dans le processus, soit environ 152 Mo sur la Gironde, donc 4. L'index de C (42,3 Mo) est rangé sur le serveur de la base et n'ajoute rien au processus du site, 5.
- Lisibilité : A demande une seule idée, la distance d'édition, 4. B demande la règle des trigrammes, l'index inversé et la distance d'édition qui départage les dix candidats, 2. C demande la règle des trigrammes, l'extension SQL `pg_trgm` avec son index GiST et la même distance d'édition, 2.
- Robustesse : les trois tests des adresses passent (les deux échecs de `npm test` relèvent des ateliers 9 et 10). A répond « 110 rue heron » au ticket T-0488 et n'a été mesurée que sur 10 saisies, 2. B et C retrouvent 288/300 (96,0 %) et ratent les mêmes : 9 saisies sans numéro (ambiguës par nature), 2 fautes sur 24, 1 commune sur 4 ; tous deux rendent « 2 rue sainte catherine » sur T-0488, 4.
- Coût : A ne change rien, 5. B, ce sont quelques lignes et un index reconstruit à chaque chargement du référentiel (2,0 s sur 229 329 adresses), 3. C exige une extension à activer, une table et un index à entretenir dans la base (construction en 9,1 s), 2.

## 3. La sensibilité

Avec temps 5 et lisibilité 1 (mémoire 1, robustesse 3, coût 2) : A 30, B 49, C 43. B reste devant.

En gardant les autres poids, A vaut poids du temps + 25, B vaut 5 × poids du temps + 24 et C 4 × poids du temps + 23 : B passe devant A dès que le temps pèse quelque chose, et devant C quel que soit ce poids (B − C = poids du temps + 1). La décision ne bascule donc pas sur le temps. C ne passe devant B que si la mémoire pèse plus que le temps et le coût réunis (plus de 5).

Point fragile : si le PostgreSQL d'Efrei descendait sous 0,1 s par saisie sur la Gironde, C prendrait 5 en temps et ne serait plus qu'à un point de B (39 contre 38). Ce temps n'a pas été mesuré, à dire en rubrique 5 de l'ADR.

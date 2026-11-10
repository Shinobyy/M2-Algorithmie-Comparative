# ADR-000 - Titre de la décision (une ligne, à l'infinitif : "Retrouver un colis par son numéro de suivi")

Une page. Six rubriques, dans cet ordre. Copiez ce fichier en `ADR-00N.md` et remplacez chaque bloc.

## 1. Contexte

Le ticket du jour, les volumes, la contrainte qui compte. Trois à cinq lignes.

Exemple : la page "Où est mon colis" cherche un numéro de suivi parmi les événements de scan du jour, 70 000 lignes en septembre 2026, 1 200 000 sur un mois glissant. Elle est appelée 40 000 fois par jour. La contrainte est le temps de réponse par requête, pas le temps de construction.

## 2. Options comparées

Deux au moins, la version naïve du legacy comprise. Une ligne par option, avec la complexité annoncée.

- Option A (legacy) : ce que fait le code actuel. Complexité annoncée : O(...).
- Option B : ... Complexité annoncée : O(...).
- Option C : ... Complexité annoncée : O(...).

## 3. Mesures

Le protocole en cinq lignes, puis le tableau, puis la courbe.

- Machine : "Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz" memoire=15Go systeme="Linux 7.0.0-38-generic", Node v22.19.0
- Jeux : sujet suivi-construction, 10 000, 100 000, 996 844.
- Répétitions et échauffement : 5 répétitions, 3 échauffement.
- Statistique retenue : la médiane.
- Ce qui est mesuré : temps total, mémoire et ce que la production paiera si ce n'est pas le temps de votre poste (requêtes à la base, appels d'API, `npm run couples`).

| Option | 10 000 | 100 000 | 996 844 | Mémoire |
| --- | --- | --- | --- | --- |
| A Tableau | ... ms | ... ms | 148,3209 ms | ... Mo |
| B Map | 3.73 ms | 38.49 ms | 1140.93 ms | ... Mo |
| C Set | ... ms | ... ms | 148,3209 ms | ... Mo |

Courbe : `bench/resultats/<horodatage>-<sujet>.svg` (générée par `npx tsx bench/courbe.ts <csv>`).

## 4. Décision

Une phrase : l'option retenue et le critère qui a tranché.

## 5. Conséquences et limites

Ce que la décision coûte (mémoire, code, dépendance), ce qui n'a pas été mesuré. Trois à cinq lignes.

Domaine de validité : les paramètres que vous avez fait varier et leurs valeurs (volume, livreurs, hubs, doublons), ceux que vous n'avez pas fait varier, et à partir de quel volume la décision ne tient plus. Une à trois lignes, avec un nombre.

## 6. En trois phrases pour la directrice des opérations

La même décision, sans un seul terme technique. Elle doit pouvoir la répéter à un livreur.

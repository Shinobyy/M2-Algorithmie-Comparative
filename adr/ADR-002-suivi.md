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
- Jeux : jeu grande, sujets suivi (100 requêtes, 4 sur 5 existantes, 1 mal tapée) et suivi-construction, 10 000, 100 000, 996 844 scans.
- Répétitions et échauffement : 5 répétitions, 3 échauffement.
- Statistique retenue : la médiane.
- Ce qui est mesuré : temps de 100 requêtes sur la structure déjà construite (sujet suivi), mémoire de la structure construite (sujet suivi-construction). Par requête projetée sur six mois : médiane à 996 844 / 100, × 13 pour tableau et Set (qui relisent tous les scans), Map inchangée.

**Requêtes** (sujet suivi, temps de 100 requêtes sur la structure déjà construite) :

| Option | 10 000 | 100 000 | 996 844 | Par requête sur six mois |
| --- | --- | --- | --- | --- |
| A Tableau | 40,17 ms | 365,71 ms | 3 348,77 ms | environ 435 ms |
| B Map | 0,065 ms | 0,077 ms | 0,071 ms | environ 0,0007 ms |
| C Set | 13,94 ms | 224,40 ms | 2 108,66 ms | environ 274 ms |

**Construction** (sujet suivi-construction, une fois à l'arrivée des scans, à 996 844 scans) :

| Option | Temps | Mémoire |
| --- | --- | --- |
| A Tableau | 0,03 ms | environ 1 Ko |
| B Map | 1 140,93 ms | 7 172 Ko |
| C Set | 451,45 ms | 5 122 Ko |

CSV : `bench/resultats/2026-11-10T09-40-26-suivi.csv` et `bench/resultats/2026-11-10T09-43-32-suivi-construction.csv`.

Courbe : `bench/resultats/2026-11-10T09-40-26-suivi.svg`

## 4. Décision

L'option retenue est la Map, et le critère qui a tranché est le temps de réponse par requête.

Domaine de validité : Les paramtères qui ont été fait varier sont le volume (10 000, 100 000, 996 844), le nombre de livreurs (1), le nombre de hubs (1), et la présence de doublons (aucun). Les paramètres qui n'ont pas été fait varier sont le type de requête (suivi) et la structure de données (Map). La décision ne tient plus à partir d'un volume supérieur à 1 000 000 scans.

## 6. En trois phrases pour la directrice des opérations

Nous avons testé trois façons de retrouver un colis par son numéro de suivi. La meilleure façon est celle qui utilise une carte (Map) pour stocker les informations, car elle est beaucoup plus rapide que les autres méthodes. Cette méthode fonctionne bien tant que le nombre de colis à suivre reste inférieur à un million.

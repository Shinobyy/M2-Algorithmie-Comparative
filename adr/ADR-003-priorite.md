# ADR-003 - Servir les colis les plus urgents d'abord sur le quai

## 1. Contexte

Le soir, les colis arrivent sur le quai et les préparateurs prennent toujours le plus urgent : les pharmacies d'abord, puis les express, puis le standard ; à priorité égale, par fin de créneau, puis par heure de création, puis par ordre d'arrivée. La file du legacy (2019) est retriée en entier à chaque arrivée. Le quai a reçu 9 000 colis le 9 novembre et en attend 15 000 par soir en décembre. La contrainte est double : tenir ce volume, et ne jamais servir un colis avant un plus urgent.

## 2. Options comparées

- Option A (legacy) : la file est retriée en entier à chaque arrivée et décalée à chaque prise. Complexité annoncée : O(n log n) par arrivée et O(n) par prise, soit O(n²) au moins sur la soirée.
- Option B (tas binaire) : le plus urgent reste au sommet, le rang d'arrivée départage les égalités complètes. Complexité annoncée : O(log n) par arrivée et par prise.
- Option C (file à seaux) : une file par priorité, servie dans l'ordre d'arrivée. Complexité annoncée : O(1). Elle suppose que les colis arrivent dans l'ordre de leur création et qu'une priorité n'a qu'une fin de créneau.

## 3. Mesures

- Machine : Intel Core i7-8650U, 8 cœurs, 15 Go, Linux 7.0.0-38, Node v22.19.0.
- Jeu : `data/jeux/grande`, soirées de 1 000, 9 000 et 15 000 colis arrivés dans l'ordre de leur création.
- Répétitions : 5, échauffement : 2.
- Statistique retenue : la médiane.
- Mesuré : durée de la soirée complète (arrivées et prises) et pic mémoire, sur le processus de Quai.

| File | 1 000 | 9 000 | 15 000 | Mémoire à 15 000 |
| --- | --- | --- | --- | --- |
| A legacy | 40,7 ms | 3 019,9 ms | 8 393,5 ms | 10,8 Mo |
| B tas | 3,8 ms | 28,6 ms | 49,0 ms | 2,1 Mo |
| C seaux | 0,5 ms | 2,4 ms | 5,4 ms | 0,9 Mo |

Les trois files rendent la même somme de contrôle à chaque taille. Épreuve : quand les colis arrivent dans l'ordre du fichier et non dans celui de leur création, le tas garde la somme de contrôle du legacy (89c52d41 à 1 000 colis), la file à seaux la perd dès 1 000 colis (bc776f41).

Mesures : bench/resultats/2026-11-10T11-12-56-priorite.csv et bench/resultats/2026-11-10T11-13-31-priorite-arrivees.csv

Courbe : bench/resultats/2026-11-10T11-12-56-priorite.svg

Grille : `adr/GRILLE-ADR-003.md`, A 36, B 40, C 35.

## 4. Décision

L'option retenue est la B, le tas binaire : 170 fois plus rapide que le legacy à 15 000 colis, et c'est la seule option rapide qui sert les colis dans le bon ordre quel que soit l'ordre d'arrivée sur le quai.

## 5. Conséquences et limites

Le tas ajoute une structure nouvelle pour l'équipe (`bench/variantes/tas.ts`, 59 lignes, sans dépendance) et une Map des rangs d'arrivée, à comprendre pour maintenir le code. La file à seaux, dix fois plus rapide, est écartée parce qu'elle ignore la fin de créneau et l'heure de création : elle ne redeviendrait possible que si l'ordre d'arrivée sur le quai était garanti.

Domaine de validité : mesuré de 1 000 à 15 000 colis par soirée, sur un seul quai, doublons de numéro de suivi écartés, sur une seule machine. À 15 000 colis, la soirée coûte 49 ms ; en O(n log n) elle reste loin de la seconde bien au-delà de 100 000 colis, ce qui n'a pas été mesuré. Une priorité inconnue n'est rejetée par aucune option, le tas compris.

## 6. En trois phrases pour la directrice des opérations

La nouvelle file du quai fait le travail de toute une soirée de décembre en un vingtième de seconde, contre plus de huit secondes aujourd'hui.
Elle sert toujours les pharmacies en premier, même quand les colis arrivent sur le quai dans le désordre.
L'option encore plus rapide a été écartée, car elle faisait passer certains colis avant de plus urgents.

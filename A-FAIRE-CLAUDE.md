# Passation pour l'autre compte Claude (16 novembre 2026, soir)

Lis d'abord `CLAUDE.md`, puis `PROJET.md` (section « Tests de 9/10, 2e tour »), puis ce fichier.
Règles : récupérer la dernière version (`git pull --rebase origin main`) juste avant de modifier, petits commits, `CACHE` +1 dans `sw.js` à chaque modification de l'app, push sur `main`. Ne pas changer les exercices du programme de Nils sans lui demander. Agents de test : utiliser le modèle **Sonnet** (Opus consomme trop).

## Objectif de Nils
Arriver à **10/10** chez tous les testeurs (6 pros + 12 utilisateurs) en appliquant leurs conseils, puis refaire tester.

## Notes actuelles (2e tour, avant les dernières corrections)
- Pros : préparateur 8, kiné 8,5, diététicien 7,5, coach force 7,5, coach femmes/maison/street 7,5, directeur artistique 7,5.
- Utilisateurs : Léa 8, Monique 7, Enzo 7, Sarah 7,5, Karim 7,5, Chloé 7, Thomas 8, Julie 8, Mehdi 8, Yanis 6,5, Paul 7,5, Nils 8,5.
- Personnages et rapports détaillés : voir la section du 2e tour dans `PROJET.md`. (Les rapports complets étaient dans le scratchpad de l'autre session, ils ne sont pas dans le dépôt.)

## Fait ce soir (déjà en ligne)
- Retours de Nils après sa 1re séance : charges rondes (multiples de 5 sur machines et poulies dès 20 kg, fonctions `round5`/`nice` + `plan()` qui enveloppe `plan0()`), charge modifiée gardée pour les séries suivantes et après validation (`DRAFT`), heure de reprise dans le chrono (`startRest`), calcul des disques (`platesTxt`), journal des séances dans Progrès avec correction (`journal()`, `daySheet()`).
- Animations : 15 mouvements redessinés ou ajoutés (bloc « animations ajoutées le 16 nov. » avant `ANIMUID`).
- Générateur : variété maison/dehors, pas deux exercices de la même famille par séance (`FAMX`), poussée ≤ 1,25 × tirage (étape 6d), pas de squats sautés pour muscle/force/ados, question « Ta journée » (`p.act`) dans l'étape `body` pour les calories.
- Image presse pectorale refaite.

## À faire, dans l'ordre
1. **Images à refaire** (retirées de `MREADY` en attendant, l'animation s'affiche à la place) : `seatedRow` (rowing assis, coussin contre la poitrine, poignées tirées vers soi), `preacherCurl` (pupitre incliné + siège + barre EZ), `machineCurl` (curl biceps à la machine, coussin pour les bras + poignées qui montent vers le visage). Même style que les autres (fond noir, lumière rouge, sans personne, sans logo : voir `PROMPTS-IMAGES.md`). Cloudflare Workers AI (flux-2-dev) gratuit ~10 000 neurones/jour, quota remis à zéro à minuit UTC ; Pollinations gratuit ne marche plus (402 ou images floues). Vérifier aussi `legCurl`, `hipThrust`, `legPress` (un peu ambigus). Chaque image validée : post-traitement 640×640 avec vignette, clé remise dans `MREADY` (index.html) et `CORE` (sw.js).
2. **Plus d'exercices débutants sans matériel** pour que A, B, C ne se ressemblent pas (maison et dehors : variantes de squat, fentes arrière, step-up, hip thrust sur canapé, pompes genoux…).
3. **Parcours « skills » street workout** niveau 2-3 (muscle-up, front lever, planche, poirier) avec paliers.
4. **Ischios** : encore absents dans certains programmes maison niveau 0 avec mal au dos ; superman au plus 1 fois par semaine (vérifier).
5. **Questionnaire en thème clair** (`.onb` est toujours sombre, couleurs `#F5F5F7` en dur).
6. **Mode simple** (gros texte, un exercice à la fois, moins de texte en séance).
7. **Nutrition concrète** : journées types et liste de courses selon les calories, allaitement (+400 à 500 kcal), protéines par repas.
8. **Ressenti par série (RPE/RIR)** et incréments réglables (1,25 kg, haltères par paire).
9. **Courbe de la force estimée** dans la fiche exercice de Progrès, séparation lourd/léger.
10. **Poids du sac à dos** saisissable pour les exercices lestés.
11. **Format circuit** pour les séances de 30 min maison/dehors.
12. **Côté jeu** : badges, défis, bilan du mois, message gentil après une absence ; import Hevy/Strong (CSV).
13. Ensuite : relancer un tour de tests (1 agent Sonnet, jury de 6 pros + 12 utilisateurs, sur une copie figée servie en local), corriger, recommencer jusqu'à 10/10.

## Questions en attente pour Nils
- Veut-il que la séance A commence par le dos (son point faible) au lieu des pecs ?
- Quels exercices avaient des explications pas claires pendant sa séance ?
- Chrono façon Apple (en petit en haut, hors de l'app) : impossible en app web sur iPhone, il faudrait une vraie app.

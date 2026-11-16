# Bilan des 6 experts (15 novembre 2026)

Six experts ont audité la version publiée le 15 novembre (commit `63d31da`), en lisant le code et en testant des dizaines de profils avec Playwright. Ils n'ont rien modifié.
Les rapports complets, avec chaque correction exacte (valeurs, textes à écrire) et sa source ou la mention « avis d'expert », sont dans le dossier `experts/` :
- `experts/prep.md` : préparateur physique (programmation, volume, progression, 2 objectifs)
- `experts/kine.md` : kinésithérapeute du sport (technique, douleurs, ados, seniors, santé)
- `experts/nutri.md` : diététicien du sport (calories, protéines, poids)
- `experts/femmes.md` : coach femmes et débutantes
- `experts/maison.md` : coach maison et street workout
- `experts/force.md` : coach force pour confirmés

**Mission du compte Claude qui reprend : appliquer ce bilan**, dans l'ordre ci-dessous (dangereux d'abord). Pour chaque point, lire la section correspondante du rapport cité avant de coder. Respecter les règles de `CLAUDE.md` et `PROJET.md` : programme figé, rien changer au programme de Nils sans lui, tests Playwright, `CACHE` +1 dans `sw.js`, petits commits. Cocher chaque case ici quand c'est fait et publié.

---

## 1. DANGEREUX : à corriger en premier

### Calories et poids (nutri, prep, femmes, kine : les 4 l'ont trouvé)
- [x] **Jamais de déficit de calories avant 18 ans** : l'objectif « perdre du gras » est bloqué, ou remplacé par « bouger plus et bien manger » sans baisse de calories (ex. actuel : fille de 14 ans, 45 kg → 1 600 kcal). → nutri, prep, femmes
- [x] **Jamais de déficit sous un IMC de 18,5** (et pas de baisse sous un IMC de 20 selon femmes.md) : objectif « perdre du gras » bloqué avec une phrase bienveillante, proposition de « muscle » ou « forme ». → nutri, prep, femmes
- [x] **Plancher de calories** : jamais sous le métabolisme de base (ex. actuel : femme de 60 ans à 1 100 kcal). → nutri, prep, femmes
- [x] **Alerte si le poids baisse de plus de 1 % par semaine** (moyenne sur plusieurs pesées) ; parler de l'arrêt des règles comme signal d'alarme chez les femmes. → nutri

### Âge et sécurité (kine, maison, prep)
- [x] **Moins de 18 ans** : retirer les exercices lourds à la barre (soulevé de terre, squat barre…) et les charges hautes (ex. actuel : garçon de 13 ans, soulevé de terre 3 × 10-15 à 55 kg). Le questionnaire accepte dès 12 ans : revoir l'âge minimum et les règles. → kine
- [x] **Plus de 55 ans** : retirer les sauts (squats sautés à 60 ans), ajouter équilibre et consignes adaptées. → kine, maison
- [x] **Questions de santé avant de commencer** : cœur, tension, malaise à l'effort, grossesse ou post-partum, opération récente → message « demande l'avis de ton médecin / ta sage-femme » et exercices adaptés. → kine, femmes
- [x] **Grossesse et post-partum** : une case dédiée ; retirer crunchs, planche, hip thrust sur le dos et tout ce qui est contre-indiqué ; renvoi vers la sage-femme. → femmes, kine

### Bouton « version plus dure » (kine, maison)
- [x] Il doit **tenir compte des douleurs, de l'âge, du niveau et du matériel disponible** (ex. actuel : femme de 60 ans, genoux douloureux, sans chaise → squat sur une jambe ; dehors sans barres → dips aux barres). Ne jamais proposer une version qui change de muscle ou demande du matériel absent. → maison, kine

### Exercices et matériel (kine, maison)
- [x] Consignes de sécurité manquantes : développé couché à la barre (butées, pareur, pas de rebond), mollets à la presse (pied qui glisse), chaise, table, barre de porte, sac à dos lesté (« sac devant toi » pour les tractions est risqué). → kine, maison
- [x] Dips sur chaise : ne plus les donner aux débutants ni aux plus de 60 ans, ni comme « version plus facile » des dips. → maison

### Charges des confirmés (force)
- [x] **Charges de départ squat et soulevé de terre** : ne plus les déduire de la presse (300 kg de presse → 150 kg au squat). Demander la charge actuelle, sinon partir prudemment. → force
- [x] **Un historique par façon de faire** : un même exercice fait en lourd (3-5 reps) et en léger (8-12) doit avoir deux historiques, sinon le jour léger repropose la charge lourde. → force

---

## 2. FAUX : à corriger ensuite
- [ ] **Programmes déséquilibrés** : 75 combinaisons débutantes sur 378 n'ont aucun exercice de dos ou aucun développé dans la semaine. Cause : le plafond de 6 exercices (ajouté le 15 nov.) retire des gros exercices qui ne sont pas une priorité. Il faut garantir au moins un tirage et une poussée par semaine (et l'équilibre poussée/tirage). → femmes, maison
- [ ] **Durée choisie toujours pas tenue** : 30 min → environ 45, 45 → environ 62. → prep
- [ ] **Plafond de volume selon le niveau** (ex. : débutante 5 jours → 93 séries par semaine ; l'objectif « forme » reçoit plus de volume que « muscle »). → prep
- [ ] **Pectoraux à 6 séries par semaine en haut/bas**, même en priorité. → prep
- [ ] **Perte de gras** : répétitions trop hautes, repos trop courts, textes « brûler » et « repos courts, tu brûles plus » faux. → prep, femmes, nutri
- [ ] **Douleur au genou** : les cuisses sont presque supprimées alors que les renforcer fait partie du traitement ; garder des variantes douces. Rowings penchés malgré le mal de dos, triceps au-dessus de la tête malgré les épaules, « chaise contre un mur, doux pour les genoux » inexact. → prep, kine
- [ ] **Force** : chaque grand mouvement une seule fois par semaine ; squat goblet et haltères en 5-8 reps ; toujours 5-8 toute l'année sans travail plus lourd ni périodisation ; repos coupés à 90 s sur les gros exercices. → prep, force, femmes
- [ ] **Baisses de charge** trop petites (70 kg × 4 → 67,5 kg × 8, l'app tourne en boucle sur l'échec) : recalculer depuis le RM estimé (−5 à −10 %) et viser le bas de la fourchette. Séries à charges différentes mal lues. → force
- [ ] **Progression au poids du corps** : tractions négatives et tractions qui se renvoient en boucle ; retour à la version facile dès une série ratée ; lest des tractions et dips qui n'augmente jamais ; niveau d'élastique non noté ; leg curl à la serviette proposé dehors ; table ou sac prescrits sans confirmation ; aucun échauffement à la maison et dehors ; objectif force sans effet sans charge. → maison, force
- [ ] **Barre** : une charge de 15 kg proposée alors que la barre seule pèse 20 kg. → femmes
- [ ] **Priorité fessiers débutante** : seulement 2 séries par exercice. → femmes
- [ ] **Superman compté comme travail du dos** ; 14 séries de cuisses dans une séance dehors. → prep
- [ ] **Rythmes de poids** en kg fixes au lieu de % du poids ; alerte « ça monte vite » identique pour tous les objectifs ; accueil de Nils : alerte sur une seule pesée ; calories en plus données aux personnes en surpoids qui veulent du muscle ou de la force. → nutri
- [ ] **Textes** : « guidé pas à pas » (mode qui n'existe pas), « Personne ne te regarde » affiché à la maison, textes au masculin. → femmes
- [ ] **Plus de 55 ans forcés en 8-12 même au niveau 3** ; 6 jours demandés donnent 5 séances sans prévenir ; pas de dips en salle. → force

---

## 3. Décision : 2 objectifs combinés (préparateur physique + diététicien)
Le propriétaire veut pouvoir choisir 2 objectifs, certains combinables, d'autres non. Décision des experts :
- **Au maximum 2 objectifs**, un **principal** (il décide des calories et des répétitions) et un **secondaire**.
- **Autorisés** : Muscle + Force, Gras + Force, Gras + Forme, Gras + Muscle (seulement niveau 0-1 ou IMC ≥ 25).
- **Interdits**, choix grisé avec une phrase simple : Forme + Muscle et Forme + Force (le muscle et la force remettent déjà en forme), Gras + Muscle pour un confirmé mince.
- **« Perdre du gras » bloqué** avant 18 ans et sous un IMC de 18,5.
- **Nutrition** : les calories suivent l'objectif le plus prudent, les protéines prennent la fourchette la plus haute.
- Dosage exact de chaque combinaison (répétitions, volume, cardio) : `experts/prep.md` ; calories et protéines avec le code : `experts/nutri.md`.

---

## 4. AMÉLIORABLE (après le reste)
- Partie douleur du Guide plus complète (courbature ou blessure, signes d'alerte, respiration), partie seniors avec équilibre. → kine
- Plafond de protéines à revoir, créatine en option pour les adultes (jamais avant 18 ans), éviter le mot « brûler », pouvoir cacher son poids. → nutri, femmes
- Règles, variations de poids, idées reçues (« je vais devenir massive », « les abdos font fondre le ventre »). → femmes
- Alerte si une débutante se construit une séance trop longue (85 min). → femmes
- Ordre des exercices, échauffement plus complet (plus de 2 séries pour les charges lourdes), montée de charge après « À l'échec », semaine légère comptée au calendrier, stagnation. → prep, force
- Ischios pour les débutants sans matériel, équilibre poussée/tirage à la maison, volume trop haut chez les confirmés et trop bas chez les seniors, test du nombre de pompes et de tractions. → maison
- Confirmés : RPE par série, RM estimé, records complets, incréments réglables, bouton « dupliquer la séance » dans le constructeur (73 gestes pour 4 séances), tirage vertical oublié par le constructeur, séries sous 5 reps, plus d'exercices à la barre (roumain, squat avant…), export CSV, calcul des disques. → force

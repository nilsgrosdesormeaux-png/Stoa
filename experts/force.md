# Stoa : audit « confirmé force / hypertrophie »

Point de vue : pratiquant de 3 ans et plus (powerlifting ou bodybuilding naturel) qui connaît ses séances. Il veut surtout suivre sa progression, ou un programme vraiment avancé.
Tests : 11 profils confirmés passés dans `generate()` (`gen.txt`) ; un programme Haut/Bas de 4 séances construit à la main sur iPhone 13, puis 5 séances jouées en avançant l'horloge (`ui.txt`, captures `01` à `05.png`) ; cas limites de `plan()` (`plan.js`).
Fichiers de test : `gen.js`, `ui.js`, `plan.js`, dans ce dossier.

**Verdict :** une bonne app de **double progression pour un intermédiaire en hypertrophie**. Pour la **force**, ce n'est pas encore un outil de confirmé : on ne peut pas descendre sous 5 répétitions, il n'y a ni RPE, ni RM estimé (e1RM), ni série lourde suivie de séries allégées, et un même exercice n'a qu'un seul historique. La logique de baisse de charge est fausse. Face à Hevy ou Strong, il manque surtout le suivi : e1RM, records complets, graphiques, export CSV.

---

## DANGEREUX

### D1. Charges de départ à la barre calculées depuis la presse à cuisses
- **Constat :** un profil qui déclare presse 300 kg, développé 120 kg et tirage 90 kg reçoit **squat 150 kg en 4×5-8, soulevé de terre 175 kg en 4×5-8, hip thrust 150 kg et développé militaire 72,5 kg**. L'app affiche « Première fois : commence à 150 kg ».
- **Pourquoi c'est faux :** le rapport presse/squat varie de 2 à 4 selon la machine et la personne. `factorOf` applique pourtant le même facteur, linéaire, à tous les gros exercices de la famille « jambes ».
- **Correction :**
  - si le niveau est 2 ou plus et qu'il y a une barre, demander directement les charges actuelles à la barre (squat, développé couché, soulevé de terre, militaire : charge × répétitions) ;
  - sinon, pour `unit==="barre"`, prendre au plus 60 % de l'estimation, avec le message « monte par paliers de 10-20 kg jusqu'à une série à 2-3 répétitions en réserve » ;
  - ne jamais dériver le soulevé de terre de la presse.
- **Source :** avis d'expert. Le principe de spécificité des tests de 1RM est bien établi (Buckner 2017, *Sports Med*, « Specificity of strength »).

### D2. Un même exercice partage un seul historique entre deux séances aux schémas différents
- **Constat :** l'historique est rangé par `uid`. Après un jour lourd au squat (160 kg × 3), le jour « volume » en 8-12 propose **160 kg × 4/4/4** avec « Même charge, une répétition de plus ». Celui qui suit la charge affichée en visant la fourchette 8-12 se retrouve à 160 kg pour 8 répétitions. Cela touche toute programmation ondulante (DUP) et les programmes Haut/Bas où le même mouvement revient deux fois.
- **Correction :** ranger le journal par `uid + schéma` (par exemple `uid@min-max`, ou un `slot` par séance). Le RM estimé (e1RM) resterait commun pour les records et les graphiques. Variante : calculer la charge proposée à partir du RM estimé et du schéma (voir F1).
- **Source :** avis d'expert ; la programmation ondulante est décrite par Zourdos 2016, *JSCR*.

---

## FAUX

### F1. Les baisses de charge sont trop petites pour l'objectif de répétitions fixé
- **Constat :**
  - développé militaire à 70 kg × 4/4/3/3 deux fois : l'app propose **67,5 kg × 8/8/8/8**, soit −3,5 % de charge pour le double de répétitions ;
  - développé couché à 100 kg × 4/4/3/3/3 deux fois « à l'échec » : l'app propose **97,5 kg × 6**.
- **Pourquoi c'est impossible :** 70 kg × 4 correspond à un RM estimé (e1RM) d'environ 79 kg (Epley). Or 67,5 kg × 8 demande un e1RM d'environ 85 kg. L'utilisateur échoue de nouveau, et l'app tourne en boucle.
- **Correction :** dans `down()` et les deux règles de baisse, calculer :
  - `e1 = kg×(1+reps/30)` sur la meilleure série ;
  - nouvelle charge = `e1 / (1+(min+2)/30)`, arrondie à l'incrément, soit le bas de la fourchette à 2 répétitions en réserve (environ −5 à −10 %) ;
  - objectif de répétitions = `e.min`, pas `e.max` ni `e.max-2`.
- **Source :**
  - formule d'Epley (1985) ;
  - le retrait de 5 à 10 % est la pratique courante (avis d'expert) ;
  - RPE fondé sur les répétitions en réserve : Zourdos 2016, *JSCR* 30(1).

### F2. Charges différentes d'une série à l'autre : la charge retenue est celle qui revient le plus souvent
- **Constat :**
  - 100×5, 100×5, 95×6, 95×6 → l'app propose **100 kg × 6/6/7/7**, donc plus de répétitions à une charge plus lourde ;
  - série lourde 200×3 puis 3×6 à 170 → l'app propose **170 kg × 4/6/6/6** et ignore la série lourde.
- **Correction :**
  - proposer série par série (`out.kg` en tableau, ou au moins `S[j].kg`) ;
  - ajouter un type d'exercice « série lourde + séries allégées » : une série lourde visée à RPE 8, puis des séries à −10 %.
- **Source :** avis d'expert ; c'est le standard des programmes avancés de type RTS ou 5/3/1.

### F3. Tractions lestées : on ne peut pas progresser en lest
- **Constat :** à 25 kg × 8×3 (haut de la fourchette), l'app répond « Ralentis la descente (3 secondes) », alors que la colonne « Lest » existe.
- **Correction :** pour `weightedPull` et `weightedDip` (ou tout `bw` avec `kg>0`), appliquer la double progression sur le lest, avec +2,5 kg en haut de fourchette.
- **Source :** avis d'expert.

### F4. Objectif force : rien sous 5 répétitions et aucune périodisation
- **Constat :** tout le programme reste en 5-8 répétitions (≈ 75-85 % du 1RM), toutes les semaines, de la semaine 1 à la 5. Pas de travail en 1-5 répétitions au-delà de 85 %, ni de bloc (accumulation, intensification, pic).
- **Correction :** pour `goal==="force"` et `lvl>=2` :
  - les 3 ou 4 mouvements principaux (squat, développé couché, soulevé de terre, militaire) suivent une **ondulation sur 3 semaines** : 4×6 à RPE 7, puis 5×4 à RPE 8, puis 4×3 à RPE 8-9 ; la semaine 6 reste la semaine légère ;
  - les gros exercices secondaires restent en 6-10, les petits en 8-15.
- **Source :**
  - Schoenfeld 2014, *JSCR* 28(10) : 7×3 contre 3×10, force supérieure avec les charges lourdes ;
  - Schoenfeld 2017, *JSCR* 31(12), méta-analyse charges lourdes contre légères : 1RM meilleur au-delà de 60 % ;
  - Harries 2015, *JSCR* : la périodisation n'est pas inférieure, et elle est utile au confirmé.

### F5. Le « gros exercice » sert de règle universelle
- **Constat :** squat goblet 4×5-8, soulevé de terre jambes tendues aux haltères 34 kg en 4×5-8, rowing un bras 20 kg en 4×5-8, tirage vertical et hip thrust en 5-8 avec 3 min de repos, chez un homme confirmé de 85 kg.
- **Pourquoi c'est faux :** le squat goblet et le soulevé de terre aux haltères ne se chargent pas assez lourd pour 5 répétitions. La prise et le haut du dos lâchent avant les jambes.
- **Correction :**
  - réserver les fourchettes de force à `unit==="barre"` (et à la presse, au hack squat, à la Smith) ;
  - les haltères et les poulies en `comp` passent en 8-12 avec 2 min de repos ;
  - exclure le squat goblet si `lvl>=2` et qu'une barre ou une presse existe.
- **Source :** avis d'expert.

### F6. Squat et soulevé de terre une seule fois par semaine pour l'objectif force
- **Constat :** en Haut/Bas sur 4 jours et sur 5 jours, le squat à la barre ne revient qu'une fois. Le Bas 2 met du squat goblet ou de la presse, et le soulevé de terre ne revient qu'une fois.
- **Correction :** pour `goal==="force"`, faire en sorte que le Bas 2 ait un squat (variante à la barre, plus léger, en 3-5×3-5) et le Bas 1 un soulevé de terre ou une variante de hanche.
- **Source :** Grgic 2018, *Sports Med* 48 : une fréquence plus haute donne plus de force sur les mouvements polyarticulaires.

### F7. Après 55 ans, l'objectif force est forcé en 8-12 même pour un niveau 3
- **Constat :** les règles d'âge imposent 8-12 répétitions et 3 min 30 de repos, même au niveau 3.
- **Correction :** appliquer ces bornes seulement si `lvl<=1`. Pour un masters entraîné, garder 4-8 répétitions avec 2-3 répétitions en réserve.
- **Source :** Fragala 2019, *JSCR*, position de la NSCA sur la musculation des seniors (les charges lourdes y sont recommandées et tolérées).

### F8. Les dips n'existent pas à la salle
- **Constat :** `envTokens("salle")` ne contient pas `"dips"`, donc les dips et les dips lestés n'apparaissent jamais pour quelqu'un qui s'entraîne en salle.
- **Correction :** ajouter `"dips"` (et `"barrebasse"`) aux jetons de la salle.

### F9. « 6 jours » demandé, 5 séances données sans le dire
- **Constat :** le générateur passe directement au découpage en 5 séances (`ppl5`). Le questionnaire, lui, ne propose que 2 à 5 jours.
- **Correction :** ajouter un découpage Poussée/Tirage/Jambes ×2 (`ppl6`) pour `lvl>=2`, ou retirer le cas.

---

## AMÉLIORABLE

### A1. Ressenti trop grossier
- **Constat :** un seul ressenti par exercice, avec 3 choix.
- **Correction :** proposer en option un **RPE ou un nombre de répétitions en réserve par série** (6 à 10, par demi-points) pour les niveaux 2-3, et s'en servir dans `plan()` pour régler la charge (±2 % par point d'écart à la cible).
- **Source :**
  - Helms 2018, *Front Physiol* (RPE contre %1RM) ;
  - Larsen 2021, *Sports Med* (l'autorégulation fait au moins aussi bien).

### A2. Records et RM estimé
- **Constat :** un record, c'est seulement « kg max » ou « répétitions à la charge max ». Faire 90×10 après 100×5 n'est rien.
- **Correction :** suivre et afficher sur une courbe :
  - **le RM estimé (e1RM, Epley ou Brzycki, jusqu'à 10 répétitions)** ;
  - les records de répétitions **à chaque charge** (tableau 1 à 10 RM, comme Strong) ;
  - le record de volume par séance.
- **Source :** fonctions standard de Strong et de Hevy.

### A3. Incréments
- **Constat :** 2,5 kg fixe pour tous les exercices à la barre. Le hip thrust « machine ou barre » est à +5.
- **Correction :** un réglage d'incrément par exercice (0,5 ; 1 ; 1,25 ; 2,5 ; 5) pour les micro-disques au développé militaire et au développé couché. Par défaut, +5 kg au soulevé de terre et au squat si `wk>=140`.
- **Source :** avis d'expert.

### A4. Échauffement trop court pour une charge lourde
- **Constat :** 50 %×10 puis 75 %×5. Pour un soulevé de terre à 220 kg, cela donne 110×10 puis 165×5 : trop de répétitions au début, pas assez de paliers.
- **Correction :** une montée du type barre ×8, 40 %×5, 60 %×3, 75 %×2, 85 %×1 au-delà de 100 kg.
- **Source :** avis d'expert.

### A5. Semaine légère imposée par le calendrier, toutes les 6 semaines
- **Constat :** on ne peut ni la décaler ni la désactiver.
- **Correction :** un réglage « semaine légère : auto / toutes les N semaines / à la demande ».
- **Source :** Bell 2023, *Sports Med Open*, Delphi sur les semaines légères (pratique flexible).

### A6. Constructeur de séances : vitesse et liberté
- **Rapidité :** 73 gestes pour 4 séances (Haut/Bas). Il n'y a pas de bouton **« Dupliquer la séance »** : il a fallu refaire Haut 2 et Bas 2 carte par carte.
- **Pas de tirage vertical :** le modèle « Haut du corps » donne 2 rowings et aucun tirage vertical, car « Dos » est arrondi à 2 exercices et les deux sont des rowings. Correction : un exercice par schéma de mouvement (`vpull`, `hpull`) avant d'en ajouter un deuxième du même schéma.
- **Fourchettes de répétitions :** le minimum est **5-8**. Ajouter 1-3, 2-4, 3-5, 4-6, plus une saisie libre.
- **Repos :** il plafonne à 4 min. Ajouter 5 min.
- **Même exercice :** impossible de mettre deux fois le même exercice dans une séance (série lourde puis séries allégées, ou pause puis normal).
- **Pas d'exercice personnalisé :** il manque notamment :
  - **soulevé de terre roumain à la barre** ;
  - **squat avant (front squat)** ;
  - **développé incliné à la barre** ;
  - **développé couché prise serrée** ;
  - **soulevé de terre sumo** ;
  - **rowing Pendlay** ;
  - **good morning** ;
  - **variantes avec pause**.
- **Charges en mode « Je connais mes séances » :** l'étape « Tes charges » est sautée, alors que c'est justement le confirmé qui les connaît.
- **Source :** comparaison avec Hevy et Strong.

### A7. Ce qui manque face à Hevy ou Strong pour un confirmé
- export CSV ;
- calcul des disques sur la barre ;
- kg ou lb ;
- vrais supersets ;
- notes par série ;
- graphiques par exercice (e1RM, volume, meilleure série) ;
- historique filtrable ;
- séries « drop set » ou « échec » marquées ;
- modèles de programmes avancés (5/3/1, GZCL, PHUL, programmes RPE) ;
- chrono de repos réglable par série.

---

## Ce qui est bien
- La double progression est correcte pour l'hypertrophie : l'ajout d'une répétition marche, ainsi que les deux « facile » de suite.
- La reprise à −10 % après 14 jours d'absence.
- La semaine légère toutes les 6 semaines, à moitié des séries.
- Les gros exercices à la barre sont bien proposés en premier aux niveaux 2-3.
- 3 min de repos sur les gros exercices.
- L'écran récapitulatif du constructeur est clair, sans clavier.
- Le lest des tractions est bien reporté d'une séance à l'autre.

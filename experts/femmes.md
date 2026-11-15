# Stoa : audit « coach femmes et grands débutants »

Ce que j'ai testé : avec Playwright (iPhone 13, `#test`), 6 questionnaires faits jusqu'au bout. Léa, 22 ans, Basic-Fit, débutante complète, veut perdre du gras, priorité fessiers et ventre, a peur de mal faire. Inès, 27 ans, chez elle sans matériel. Sophie, 45 ans, reprend le sport, a mal au dos. Camille, 30 ans, veut devenir plus forte. Jade, 24 ans, en mode « je construis mes séances », avec des douleurs aux genoux. Nora, 34 ans, 78 kg, en mode « Stoa me propose, je modifie » (j'ai remplacé un exercice). Puis l'accueil, la première séance et le programme. J'ai aussi lancé `generate()` sur 12 profils féminins, et sur une grille de 1 134 combinaisons (lieu × jours × durée × objectif × niveau × 2 priorités).
Fichiers dans ce dossier : captures `*.png`, textes des écrans `*.txt`, programmes générés `gen.txt`, scripts `run.js`, `gen.js`, `grid.js`. Aucune erreur JavaScript.

**Ce qui est bon** : machines proposées en premier quand on a peur, hip thrust et abducteurs bien présents quand on choisit les fessiers. Les charges de départ sont cohérentes pour une femme : le haut du corps est à environ 50-60 % des charges d'un homme et les jambes à environ 80 %, ce qui correspond aux écarts mesurés. La règle de la première fois est bonne (« cherche une charge où tu fais au moins 10 répétitions propres »), tout comme la partie douleur, le 15 et le 112, la serviette, et le ton général bienveillant (« Pas de honte : c'est le cas de presque tout le monde au début »).

---

## DANGEREUX

### D1. Stoa conseille de manger moins à une femme déjà mince ou à une mineure
`nutrition()` applique −17 % de calories dès que l'objectif est « Perdre du gras », sans aucun garde-fou.
- 22 ans, 50 kg, 1,65 m (IMC 18,4, donc maigreur) → **1 500 kcal**, sans aucun message.
- 15 ans, 48 kg (IMC 19,2) → **1 450 kcal**. Pourtant le Guide « Moins de 18 ans » dit « mange à ta faim, pas de régime strict » : l'app se contredit.
- Les jeunes femmes sont le public le plus exposé aux troubles du comportement alimentaire et au manque d'énergie (RED-S).

**Correction** (dans `nutrition()` et sur l'écran « C'est prêt ») :
- si `age < 18` : `adj = 0` pour l'objectif gras. Texte : « Tu grandis encore : pas de régime. On mise sur le sport et des repas complets. »
- si IMC < 20 et objectif gras : `adj = 0`. Texte : « Ton poids est déjà bas : Stoa ne te propose pas de manger moins. Tu changeras ta silhouette en prenant de la force. »
- si IMC < 18,5 : ajouter « Parles-en à ton médecin avant de chercher à perdre du poids. »
- plancher dans tous les cas : `kcal ≥ métabolisme de base` (Mifflin, sans facteur d'activité).

Sources : OMS (IMC < 18,5 = maigreur) ; Mountjoy et al., *BJSM* 2023, consensus du CIO sur le RED-S ; HAS 2010, recommandations sur l'anorexie mentale (repérage chez les adolescentes).

### D2. Grossesse et post-partum : jamais demandés, programme inchangé
Une femme enceinte ou qui vient d'accoucher reçoit le même programme que les autres : crunch à la poulie, gainage, hip thrust avec une charge posée sur le ventre, et le squat sauté comme version « plus dure ». Le mot « grossesse » n'apparaît que dans une note du Guide.

**Correction** :
- à l'étape « Tu as mal quelque part ? », ajouter un choix : « Je suis enceinte, ou j'ai accouché il y a moins d'un an ».
- s'il est coché, afficher : « Bouger pendant et après une grossesse, c'est recommandé. Mais Stoa ne fait pas encore de programme adapté : demande d'abord l'accord de ta sage-femme ou de ton médecin. Après l'accouchement, fais ta rééducation du périnée avant de reprendre les abdos et les charges. »
- s'il est coché, retirer du programme : crunchs, relevés de jambes, planche, squat sauté, hip thrust à la barre, et tout exercice allongé sur le dos longtemps.

Sources : ACOG, Committee Opinion n° 804 (2020) ; OMS, recommandations sur l'activité physique (2020) ; HAS, *Rééducation dans le cadre du post-partum* (2002).

---

## FAUX

### F1. Un débutant sur cinq reçoit une semaine entière sans travail du dos, ou sans exercice de poussée
Sur la grille, **75 combinaisons sur 378 au niveau « jamais »** n'ont aucun tirage (dos) ou aucune poussée sur toute la semaine. Exemple : femme, salle, débutante, 3 × 60 min, priorités fessiers et jambes. Résultat : A, B et C contiennent presse, hip thrust, abducteurs, un développé et un crunch, mais **0 série pour le dos** sur la semaine, contre 11 séries de poussée.

Autre cas, Camille (priorités dos et bras, objectif force) : **aucun développé** sur la semaine, et un seul exercice de jambes par séance.

Ce sont les cas à 60 min ou à 2 séances qui posent problème : le plafond de 6 exercices pour un débutant (`cap`, ligne ≈ 2011) retire en dernier les gros exercices qui ne sont pas une priorité, et ce sont souvent le tirage ou le développé.

**Correction** :
- dans la boucle `while(exos.length>cap)`, ne jamais retirer le dernier exercice de poussée (`hpush`/`vpush`) ni le dernier exercice de tirage (`vpull`/`hpull`) d'une séance « Corps entier » ou « Haut du corps » ;
- retirer d'abord une 2ᵉ isolation d'une même priorité (par exemple les abducteurs quand le hip thrust est déjà là) ;
- ajouter un test : pour chaque profil, au moins 6 séries de poussée, 6 de tirage et 6 de jambes par semaine.

Source : avis d'expert. Equilibrer poussée et tirage est la base de toute programmation (NSCA, *Essentials*, 4ᵉ éd.).

### F2. Une charge de départ inférieure au poids de la barre
Profil confirmé (28 ans, 60 kg) : « Développé militaire à la barre, **15 kg** ». La barre seule pèse 20 kg (`UNIT.barre` dit bien « poids total, barre de 20 kg »).

**Correction** : pour les exercices à la barre olympique (`unit:"barre"`, sauf barre EZ), utiliser `kg0 = Math.max(20, kg0)`. Si la charge calculée est inférieure à 17,5 kg, choisir plutôt la version aux haltères.

### F3. « Repos courts : tu brûles plus »
Les repos courts ne font pas perdre plus de gras de façon notable. Ils réduisent surtout le travail fait dans la séance (et donc ce que le muscle reçoit).

**Correction** :
- carte du questionnaire : remplacer par « Des repos plus courts : la séance va plus vite. »
- `sessionIntro` gras : remplacer par « Objectif : garder ton muscle pendant que tu perds du gras. C'est surtout l'assiette qui fait baisser le gras, la séance protège ton muscle. »

Source : Schoenfeld et al., *JSCR* 2016 (repos courts = moins de progrès).

### F4. La carte « Je veux être guidé pas à pas » promet une fonction qui n'existe pas
Elle annonce « Où aller, quoi régler, comment commencer léger ». En réalité, l'app affiche le même encadré de 7 points que pour tout débutant. Pour une femme qui a peur de la salle, c'est la promesse centrale.

**Correction**, au choix :
- (a) changer le texte pour « Des conseils pour ma toute première séance » ;
- (b) mieux : quand `first===true`, faire un vrai mode guidé, un exercice à la fois, en plein écran : photo de la machine, puis « règle le siège : tes genoux sont à 90° », puis 1 série d'essai à vide, puis la charge.

### F5. « Personne ne te regarde »
Ce n'est pas vrai à la salle, et c'est absurde à la maison (le texte s'affiche aussi chez Inès). Nier la peur ne rassure pas.

**Correction** : « Commence léger. La première séance sert à apprendre les mouvements. Les autres sont concentrés sur leur séance, et tout le monde a commencé un jour. » À la maison : retirer la phrase.

### F6. La partie choisie en priorité reçoit moins de séries que le reste
Chez un débutant, les exercices simples sont réglés à 2 séries (`if(!comp&&(dur<=30||lvl===0))series=2`), même quand ils travaillent la priorité.
- Inès (priorité fessiers, à la maison) : pont fessier 2 × 12-20 et élévations de jambe 2 × 12-20, soit **4 à 6 séries de fessiers par semaine** au poids du corps, et un pont trop facile pour progresser.

**Correction** :
- `if(fpats.includes(e0.pat)) series = Math.max(series, 3)` ;
- à la maison, pour la priorité fessiers, proposer le pont sur une jambe ou le hip thrust dos sur le canapé (le niveau au-dessus du pont fessier).

Source : Schoenfeld et al., *J Sports Sci* 2017 (plus de séries par semaine = plus de muscle).

### F7. Objectif force : repos coupés à 90 s pour tenir dans la durée
Camille (5-8 répétitions) : tirage vertical avec 90 s de repos, rowing avec 150 s. La boucle qui fait tenir la séance dans le temps réduit les repos avant de retirer une isolation.

**Correction** : si `goal==="force"`, ne jamais descendre sous 150 s pour les gros exercices. Retirer une isolation à la place.

Source : Schoenfeld et al., *JSCR* 2016 ; Grgic et al., *Sports Med* 2017 (des repos de plus de 2 min favorisent la force).

---

## AMÉLIORABLE

1. **Cycle menstruel** : l'app n'en parle nulle part. Il ne faut **pas** adapter le programme au cycle (aucune preuve). Mais il faut en parler, sinon les femmes abandonnent pendant leurs règles ou en voyant leur poids monter.
   - Guide, partie Séance : « Règles : tu peux t'entraîner normalement, ça peut même soulager les douleurs. Fatiguée ou douloureuse ? Garde ta séance mais retire une série par exercice : ta progression ne bouge pas. »
   - Partie Poids : « Ton poids peut monter de 1 à 2 kg avant et pendant les règles : c'est de l'eau. Regarde la moyenne sur 2 semaines. »
   - Bouton « Séance allégée aujourd'hui » (−1 série, pour tout le monde : fatigue, mauvaise nuit, règles).
   - Sources : McNulty et al., *Sports Med* 2020 ; Colenso-Semple et al., *Front Sports Act Living* 2023.
2. **Idées reçues jamais combattues** : la peur de devenir trop musclée est la première crainte des débutantes. Ajouter au Guide :
   - « Tu ne vas pas "gonfler" : prendre du muscle est lent, et tu contrôles tout avec les charges et ce que tu manges. »
   - Quand le ventre est choisi en priorité : « Les abdos renforcent le ventre mais ne font pas fondre le gras du ventre : ça, c'est l'alimentation et l'ensemble de la séance. »
   - Sources : Roberts et al., *JSCR* 2020 (les femmes prennent autant de muscle que les hommes en proportion, et gagnent plus de force du haut du corps en proportion, ce qui est une raison de plus de ne pas négliger le haut du corps) ; Vispute et al., *JSCR* 2011 (travailler les abdos ne fait pas perdre le gras du ventre).
3. **Femmes de 40 ans et plus** (Sophie) : rien sur la ménopause, ni sur les os. Ajouter pour les plus de 40 ans : « La muscu est le meilleur allié de tes os et de tes muscles avant et après la ménopause. » Source : Watson et al., *JBMR* 2018 (essai LIFTMOR).
4. **Textes accordés au masculin** alors que l'app connaît le sexe : « Je veux être **guidé** », « Tu es **assis** / **allongé** » (une dizaine de fiches), « Tu es **prêt** pour la version plus dure », « Devenir plus **fort** », « **Motivé** ». Passer à des formulations neutres : « Je veux qu'on me guide pas à pas », « En position assise », « Sur le dos », « Tu peux passer à la version plus dure », « Gagner en force », « Motivation ».
5. **Le poids partout** : il est affiché en permanence en haut de l'écran Séance, et l'accueil demande « Note ton poids 3 fois par semaine ». C'est démotivant quand l'objectif n'est pas de maigrir, et risqué en cas de rapport difficile à la nourriture. Ajouter dans le questionnaire ou les réglages : « Je ne veux pas suivre mon poids » (cache le poids et la courbe). Pour les objectifs « forme » et « force », mettre en avant les records plutôt que le poids. *Avis d'expert.*
6. **Écran « points forts / points faibles »** : la silhouette avec « Ton ventre » ou « Tes bras » en rouge fait juger son corps. Remplacer le titre par « Qu'est-ce que tu veux renforcer en priorité ? » et les boutons par **Déjà bien / Normal / Priorité**. *Avis d'expert.*
7. **Constructeur pour une débutante** (Jade) : séance C avec 8 exercices et « environ 85 min », sans aucune alerte. `buildExo` ignore le niveau 0 (3 séries partout) et l'âge de 55 ans et plus (pas de repos plus long), et prend l'objectif « muscle » par défaut.
   - Correction : au-delà de 60 min ou de 7 exercices pour un niveau 0, afficher « C'est long pour débuter : 5 ou 6 exercices suffisent les premiers mois ».
   - Appliquer à `buildExo` les mêmes règles de niveau et d'âge que `generate()`.
8. **Basic-Fit** : « Le personnel de l'accueil peut aussi te la montrer » est souvent faux, beaucoup de clubs n'ont personne à l'accueil. Remplacer par « S'il y a un coach ou un membre du personnel en salle, demande-lui : c'est son métier. Sinon, touche l'image de l'exercice ».
9. **Abducteurs** : « il arrondit la silhouette » est un argument esthétique un peu cliché. Remplacer par « Le côté des fessiers : il stabilise le bassin et les genoux, et donne du galbe aux hanches ».
10. **Choix de douleurs incomplet** : ajouter « Aux poignets » (pompes, développés) et « Fuites urinaires à l'effort » (fréquentes chez les femmes, même sans enfant). Avec ce dernier choix : retirer le squat sauté, et conseiller la rééducation du périnée. Source : HAS (2002).
11. **Durées incohérentes** : Léa choisit 45 min, l'accueil affiche « 50 min environ ». Le même programme affiche « environ 40 min » dans l'écran de modification (Nora). Utiliser une seule fonction de calcul (`sEst`) et respecter la durée choisie à 5 min près.
12. **Objectif force pour une débutante complète** : 5 à 8 répétitions dès la 1ʳᵉ séance. Proposer 8-10 répétitions les 4 premières semaines, le temps d'apprendre les gestes, puis 5-8. *Avis d'expert* ; la force progresse avec des charges lourdes comme légères au début (Schoenfeld et al., *JSCR* 2017).
13. **Soulevé de terre jambes tendues aux haltères en 1er exercice de la séance B** chez une débutante qui a peur : c'est l'exercice le plus technique du programme. Quand on a répondu « peur de mal faire », le placer après une machine, ou proposer le banc à lombaires à 45° ou la poulie entre les jambes. *Avis d'expert.*

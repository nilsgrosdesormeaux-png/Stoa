# Audit kiné du sport : Stoa (`index.html`)

Lecture : `LIB`, `LIBB`, `P3`/`POOL`, `candidates()`, `generate()`, `nutrition()`, `plan()` (version plus dure), questionnaire (`OSTEPS`, `SWIPES`), messages de séance, Guide (`vGuide`, `guideGen`).
Tests : Playwright, `generate()` et `nutrition()` sur 15 profils (douleurs dos, genoux, épaules et les trois ensemble ; ados de 13, 14, 15 et 16 ans ; seniors de 60, 65 et 70 ans ; femme maigre). Les sorties brutes sont dans `out.txt` (même dossier).

Verdict : les consignes techniques sont dans l'ensemble justes et prudentes. Le programme de Nils (`P3`) est bien écrit. **Les gros problèmes viennent du générateur** : l'âge ne change jamais le choix des exercices, la version plus dure ne tient pas compte des douleurs, et les calories baissent pour des ados et des personnes maigres. Il n'y a aucun repérage santé avant de commencer.

---

## 1. DANGEREUX (à corriger avant de faire tester à d'autres personnes)

### D1. Ados : l'âge ne filtre aucun exercice
- **Constat** (vérifié) : à 13 ans, niveau « plus de 3 ans », objectif force, l'app donne **soulevé de terre à la barre 3 × 10-15 à 55 kg**, développé militaire debout, rowing barre et squat barre. À 15 ans dehors : tractions et dips lestés. Pourtant l'écran du questionnaire promet : « Le programme restera léger, sans charges lourdes ». `candidates()` ne lit jamais `p.age`.
- **Correction** : dans `candidates()`, si âge < 16, exclure `unit:"barre"`, `weightedPull`, `weightedDip` et `jumpSquat`. Si âge < 18, plafonner les exercices à la barre à 6-10 répétitions (jamais 15 au soulevé de terre : la technique casse avec la fatigue) et afficher sur ces exercices : « Moins de 18 ans : seulement avec un coach ou un adulte formé à côté de toi, la première fois. »
- **Source** : AAP, Stricker 2020 (*Resistance training for children and adolescents*) ; Lloyd 2014, consensus international sur la musculation des jeunes (BJSM) : la musculation est sûre **si elle est encadrée par quelqu'un de qualifié**.

### D2. Calories en baisse pour des ados et des personnes maigres
- **Constat** (vérifié) : une fille de 14 ans, 50 kg (IMC 19,5), objectif « perdre du gras » → **1 550 kcal**. Le Guide dit pourtant « Pas de régime strict ». Une femme de 22 ans, 45 kg pour 1,70 m (**IMC 15,6**, maigreur sévère) → 1 550 kcal, donc un déficit. Une femme de 45 kg pour 1,55 m → 1 200 kcal. Il n'y a aucun plancher.
- **Correction** dans `nutrition()` :
  - si âge < 18 : aucun déficit (`adj = 0`), et texte : « Tu grandis encore : pas de régime. Mange à ta faim, 3 repas et 1 ou 2 collations. Si ton poids te préoccupe, parles-en à ton médecin. »
  - si IMC < 18,5 : bloquer l'objectif « Perdre du gras » et écrire : « Ton poids est déjà bas pour ta taille : perdre du gras n'est pas conseillé. On vise plutôt la forme et la force. Si tu te trouves trop gros(se) malgré tout, parles-en à un médecin. »
  - plancher : jamais moins de 1 500 kcal (femme) ou 1 800 kcal (homme).
- **Source** : IOC, Mountjoy 2023 (déficit énergétique relatif dans le sport, RED-S) ; HAS 2010, anorexie mentale (repérage) ; OMS, seuils d'IMC.

### D3. La « version plus dure » ignore les douleurs, l'âge et le niveau
- **Constat** : `HARDER` (utilisé dans `plan()`, vers la ligne 2362) propose sans filtre `wallSit → jumpSquat`, `squatBW → bulgarian`, `chairDips → dips`, `crunch → kneeRaise`. Quelqu'un qui a coché « mal aux genoux » et fait la chaise contre le mur se voit proposer les **squats sautés**, que le filtre lui avait retirés. Un ado ou un senior aussi.
- **Correction** : avant de proposer `out.up`, vérifier `!(LIBX[up].avoid||[]).some(x=>pain.includes(x))` et la règle d'âge de D1. Remplacer aussi `wallSit:"jumpSquat"` par `wallSit:"squatBW"` : passer d'une chaise tenue à des sauts n'est pas une progression logique.
- **Source** : avis d'expert. La logique de l'app doit être la même partout.

### D4. Squats sautés pour les seniors
- **Constat** (vérifié) : à 60 ans, dehors, niveau 1, la séance C contient `jumpSquat 3 × 8-15`. Il n'y a aucun avertissement (arthrose, impact, chute).
- **Correction** : exclure `jumpSquat` si âge ≥ 55, et aussi si grossesse ou post-partum (voir F6). Pour les seniors, le remplacer par « Se lever d'une chaise vite, se rasseoir lentement » (puissance sans impact).
- **Source** : ACSM Guidelines, 11e éd. ; Fragala 2019, position NSCA sur les personnes âgées : la puissance se travaille en vitesse de montée, pas en sauts, chez le débutant âgé.

### D5. Développé couché à la barre : sécurité insuffisante
- **Texte actuel** (`barBench.reco`) : « Idéalement des sécurités, ou quelqu'un pour t'aider. »
- **Correction** :
  - `reco` : « Un banc avec des barres de sécurité (dans un rack), réglées juste sous ta poitrine. Sans sécurité et sans personne pour t'aider, fais-le à la barre guidée ou aux haltères. »
  - ajouter dans `how` : « Le pouce fait le tour de la barre (jamais la barre posée dans la paume sans le pouce). »
  - ajouter dans `how` : « Seul et sans sécurité : pas de pinces sur les disques, pour pouvoir les faire glisser si tu restes coincé. »
- **Source** : CPSC (États-Unis), décès par barre tombée sur le cou ou la poitrine au développé couché ; recommandations NSCA de parade (*Essentials of S&C*, 4e éd.).

### D6. Mollets à la presse : risque de glissade du pied (version `LIB`)
- **Constat** : le programme de Nils prévient « si ton pied glisse, arrête ». La version `LIB`, donnée à tout le monde avec des charges de départ jusqu'à 115 kg, ne le dit pas.
- **Correction**, à ajouter en tête de `how` : « Garde les sécurités de la presse en place si la machine le permet. Chaussures qui accrochent, avant du pied bien posé : si ton pied glisse, arrête tout de suite. Genoux tendus mais pas verrouillés. »
- **Source** : avis d'expert. C'est un accident classique sur presse à disques.

### D7. Aucun repérage santé avant de commencer
- **Constat** : le questionnaire ne pose aucune question sur la santé. Le seul avertissement (« cœur, tension, grossesse… demande à ton médecin ») est caché dans le Guide, partie Douleur, repliée. Une personne de 70 ans, débutante, cardiaque ou enceinte peut recevoir un programme sans rien voir.
- **Correction** : ajouter une étape « Ta santé » juste après « Tu as mal quelque part ? », avec ces cases :
  - « Un médecin m'a dit que j'ai un problème au cœur ou de tension »
  - « J'ai eu une douleur dans la poitrine, un malaise ou une perte de connaissance à l'effort »
  - « Je suis enceinte, ou j'ai accouché il y a moins d'un an »
  - « J'ai été opéré(e) ou blessé(e) il y a moins de 6 mois »
  - « Diabète traité, ostéoporose, ou autre maladie suivie »

  Si une case est cochée : « Montre ce programme à ton médecin avant de commencer. En attendant, commence très léger. » Pour les cas grossesse et post-partum, voir F6.
- **Source** : questionnaire de santé QS-Sport (arrêté du 20 avril 2017, France) ; PAR-Q+ 2023 ; ACSM 2015, Riebe (*Updated preparticipation screening*).

---

## 2. FAUX (consigne inexacte ou filtre manquant)

### F1. Rowing penché sans appui malgré une douleur au dos
- **Constat** (vérifié, profil 3 douleurs) : `bagRow` (« Debout, penché en avant… ») et `dbRearFly` (« Penché en avant, dos plat ») restent proposés à quelqu'un qui a mal au bas du dos.
- **Correction** :
  - ajouter `avoid:["dos"]` aux deux.
  - dans `dbRearFly.how[0]`, proposer la version appuyée : « Poitrine appuyée sur un banc incliné (ou penché, dos plat, si ton dos va bien). »
- **Source** : HAS 2019, lombalgie commune (adapter les positions qui provoquent la douleur) ; avis d'expert.

### F2. Extensions triceps au-dessus de la tête malgré une douleur aux épaules
- **Constat** : `overheadTri` et `dbOverheadTri` n'ont pas de `avoid`. Bras levé au maximum avec une charge : c'est la position qui réveille le plus souvent une épaule douloureuse. Elles peuvent sortir en alternative ou en 2e exercice de triceps.
- **Correction** : ajouter `avoid:["epaules"]` aux deux. Si `sw.epaulesDoux`, préférer `triPushdown`.
- **Source** : avis d'expert ; Cools 2014 (*Rehabilitation of scapular dyskinesis*) pour les positions qui provoquent la douleur.

### F3. « Chaise contre un mur… doux pour les genoux »
- **Constat** : à 90°, la chaise contre le mur est une des positions qui **chargent le plus** l'articulation sous la rotule. Ce n'est pas « doux ».
- **Correction** :
  - `how[0]` : « Dos contre le mur, descends jusqu'où ton genou reste confortable (cuisses parallèles au sol au plus bas). Si tes genoux sont sensibles, reste plus haut (genoux pliés à environ 45-60°). »
  - `pourquoi` : « Brûle les cuisses sans mouvement. Tu règles la difficulté avec la hauteur. »
- **Source** : Escamilla 2009 et Powers 2014 (contraintes sous la rotule selon l'angle du genou).

### F4. « Soulevé de terre jambes tendues aux haltères » (`rdlDB`)
- **Constat** : le nom dit « jambes tendues », la consigne dit « genoux légèrement pliés ». Un débutant va verrouiller les genoux.
- **Correction** : nom « Soulevé de terre roumain aux haltères (genoux souples) ». Ajouter dans `how` : « Les haltères restent collés aux cuisses. Arrête la descente dès que ton dos commence à s'arrondir, même si tes mains sont encore au-dessus des genoux. »
- **Source** : NSCA, *Essentials of S&C*, 4e éd.

### F5. Squat à la barre : « mains serrées » ambigu
- **Constat** : « mains serrées » peut se lire « mains rapprochées ». Mains trop proches sur la barre = épaules et coudes en contrainte.
- **Correction** : « Barre sur le haut du dos (pas sur la nuque), mains un peu plus larges que les épaules, prise ferme, coudes vers le bas. »
- **Source** : avis d'expert.

### F6. Grossesse et post-partum : rien n'est prévu
- **Constat** : aucune question, aucune adaptation. Un programme « perdre du gras » avec squats sautés, crunchs et circuit peut être donné à une femme qui vient d'accoucher.
- **Correction** : si la case grossesse ou post-partum est cochée (voir D7) :
  - **Grossesse** : exclure `jumpSquat`, `crunch`, `cableCrunch`, `kneeRaise`, `legRaise`, `lSit` et la barre lourde. Pas de déficit calorique. Afficher : « Enceinte : la muscu modérée est bonne pour toi et le bébé si ton médecin ou ta sage-femme est d'accord. Tu dois pouvoir parler pendant l'effort. Pas de blocage de la respiration, pas d'exercice avec risque de chute ou de choc au ventre. Après 4 mois, évite de rester longtemps allongée sur le dos (préfère incliné ou sur le côté). Arrête et consulte si saignement, perte de liquide, contractions, vertiges, douleur dans la poitrine ou dans un mollet. »
  - **Post-partum de moins de 6 mois** : pas de sauts ni de course avant le bilan du périnée (rééducation périnéale) et l'accord de la sage-femme ou du médecin, en général pas avant 3 mois. Pas de crunchs tant que le ventre fait un « dôme » au milieu quand tu te relèves. Afficher : « Fuites urinaires, lourdeur dans le bas du ventre ou bosse au milieu du ventre à l'effort : parles-en à ta sage-femme ou à un kiné, c'est fréquent et ça se rééduque. »
- **Source** : ACOG 2020, avis de comité n° 804 ; Mottola 2019, recommandations canadiennes (BJSM) ; Goom, Donnelly et Brockwell 2019 (reprise de la course après l'accouchement) ; HAS 2015 (post-partum).

---

## 3. AMÉLIORABLE

### A1. Partie « Douleur : quand s'arrêter » (Guide) : à compléter
Remplacer la liste par :
- **Brûlure dans le muscle pendant l'effort** : normal.
- **Courbatures** : elles arrivent 12 à 48 h après, des deux côtés, dans le muscle, et diminuent en bougeant. Normal, ça passe en 2 à 4 jours.
- **Douleur de blessure** : souvent d'un seul côté, précise (on peut la montrer du doigt), dans une articulation ou un tendon, apparue d'un coup pendant la série, ou qui augmente en bougeant. Arrête l'exercice et prends l'autre option.
- **Avec une douleur déjà connue** (dos, genou, épaule) : une gêne légère (3 sur 10 au plus) qui ne monte pas pendant la séance et revient à ton niveau habituel le lendemain matin, c'est acceptable. Si elle dépasse 5 sur 10 ou si elle est pire le lendemain, baisse la charge ou l'amplitude la fois suivante.
- **Consulte un médecin ou un kiné sans attendre 7 jours** si :
  - le genou gonfle, se bloque ou « lâche » ;
  - tu as entendu un craquement avec une douleur forte ;
  - tu ne peux plus t'appuyer sur la jambe ou lever le bras ;
  - des fourmillements ou une perte de force descendent dans un bras ou une jambe.
- **Consulte si** une douleur dure plus de 7 jours ou te réveille la nuit.
- **Urgence (15 ou 112)** :
  - douleur dans la poitrine, malaise, vertiges, essoufflement anormal ;
  - mal de dos avec perte de sensibilité entre les jambes, ou difficulté à uriner ou à se retenir ;
  - urines très foncées (couleur coca) avec des muscles très douloureux et gonflés, 1 à 3 jours après une séance très dure.

Sources : Silbernagel 2007 (modèle de suivi de la douleur) ; HAS 2019, lombalgie (signes d'alerte, syndrome de la queue de cheval) ; rhabdomyolyse d'effort (Kim 2016, revue).

### A2. Respiration : aucune règle générale
Ajouter dans « Déroulé d'une séance » : « Souffle pendant l'effort (en poussant ou en tirant), inspire en revenant. Ne bloque pas ta respiration longtemps. Si tu as de la tension, un problème au cœur, si tu es enceinte ou si tu as plus de 55 ans, ne la bloque jamais. »
Source : ACSM 2019, Pescatello (hypertension et exercice).

### A3. Seniors : pas de partie dédiée dans le Guide, pas d'équilibre
Ajouter une partie « Plus de 55 ans : tes règles », dans le même style que celle des ados :
- Échauffement de 8 à 10 minutes.
- Commence avec 2 ou 3 répétitions en réserve, ne va pas à l'échec le premier mois.
- Monte et descends des machines en te tenant.
- Pour les fentes et le squat bulgare : une main sur un mur ou un dossier.
- 2 fois par semaine, ajoute 5 minutes d'équilibre : tenir sur un pied (en te tenant au début), marcher talon contre pointe.
- Se lever vite d'une chaise et se rasseoir lentement.
- Médicaments contre la tension : relève-toi doucement après les exercices allongés.

Dans `generate()`, si `old` : ajouter un rôle « équilibre » ou donner un appui à `lunge` et `bulgarian` (vérifié : femme de 65 ans débutante → fentes arrière 4 × 8-15 sans appui).
Sources : Sherrington 2019 (Cochrane, prévention des chutes) ; Fragala 2019 (NSCA) ; OMS 2020.

### A4. Ados : douleurs de croissance
Ajouter dans « Moins de 18 ans » : « Douleur sous la rotule, sur la bosse du tibia, ou au talon, qui revient à chaque séance : c'est souvent lié à la croissance (Osgood-Schlatter, Sever). Ce n'est pas grave, mais il faut adapter : parles-en à un médecin ou un kiné. » Retirer aussi le bouton de ressenti « À l'échec » ou masquer sa progression pour les moins de 18 ans, pour rester cohérent avec « pas d'échec ».
Source : Lloyd 2014 ; avis d'expert.

### A5. Leg extension (`LIB`) : réglage
`how[0]` actuel : « Règle le dossier pour que tes genoux soient au bord du siège ». Remplacer par : « Règle le dossier pour que ton genou soit en face de l'axe de la machine (souvent un point de couleur). » C'est déjà correct dans `P3`. Pour les genoux sensibles, une version en amplitude partielle (genoux pliés de 90° à 45°) serait mieux que de retirer l'exercice. Source : Powers 2014.

### A6. Écarté à la machine (`LIB.pecDeck`)
Ajouter : « Les coudes ne passent pas derrière la ligne des épaules. » Sinon, mettre `avoid:["epaules"]`. Le texte de `P3` (« si l'épaule tire ») est meilleur.

### A7. Hip thrust et kickback : cambrure
- `hipThrust` : ajouter « En haut, ce sont les fesses qui serrent, pas le bas du dos qui se creuse : côtes basses. »
- `cableKick` : ajouter « Le bas du dos ne se creuse pas : arrête le mouvement avant. »

### A8. Élastiques et table : sécurité du matériel
- Tous les exercices `unit:"elastique"` : « Vérifie l'élastique (fissure) et l'ancrage avant chaque séance. Ne tire jamais un élastique vers ton visage : s'il lâche, il revient très fort. »
- `tableRow` : « Teste d'abord en tirant fort sur le bord sans être dessous. Jamais une table pliante, en verre ou sur roulettes. »
- Source : avis d'expert (blessures à l'œil par élastique décrites en ophtalmologie).

### A9. Développé couché aux haltères (`dbBench`)
Ajouter en fin de `how` : « À la fin, ramène les haltères sur la poitrine puis sur les cuisses en te relevant. Ne les lâche pas sur les côtés (épaules). »

### A10. Abdos et lombalgie à la salle
Avec `pain:"dos"` à la salle, `cableCrunch` (flexion du dos avec charge) passe avant `deadBug` : le bonus +4 « salle » dépasse le bonus +3 « dos ». Pour un dos douloureux, faire passer `deadBug` et `plank` en premier.
Source : HAS 2019 ; avis d'expert. Les preuves sont modestes, mais le choix prudent ne coûte rien.

### A11. Constructeur (`openBuilder`) : seniors oubliés
La ligne 3157 adapte les répétitions pour les moins de 18 ans, mais rien n'est prévu pour les 55 ans et plus (pas de repos plus long, contrairement à `generate()`). Il faut aligner les deux.

### A12. Douleurs couvertes
Seuls les genoux, le bas du dos et les épaules sont proposés. Il faudrait ajouter :
- **« Poignets »** : retirer pompes, dips, planche sur les mains ; proposer les pompes sur poignées ou les poings.
- **« Hanches »** : retirer le squat profond et le pistol.
- **« Nuque »** : retirer le crunch mains derrière la tête.

---

## Ce qui est bien (à garder)
- Les règles « jamais derrière la nuque », « ne cambre pas », « sans rebond » (curl pupitre), les sécurités de la presse et de la Smith, la planche arrêtée « quand le bas du dos commence à creuser ».
- La partie « Moins de 18 ans » (croissance non menacée, technique d'abord, 8 à 15 répétitions) : conforme aux recommandations.
- Le filtre douleurs fonctionne sur les exercices les plus à risque (squat et soulevé à la barre, fentes, dips, développé épaules), et l'option « épaules fragiles » remplace bien le développé au-dessus de la tête.
- L'échauffement (5 min + 50 % × 10 + 75 % × 5) est correct pour l'adulte. Il est un peu court pour les plus de 55 ans (voir A3).
- La reprise à −10 % après 2 semaines d'arrêt et la semaine légère toutes les 6 semaines.

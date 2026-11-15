# Audit Stoa : préparateur physique

Méthode : 43 profils générés avec `generate()` et `nutrition()` sous Playwright (sexe, âges 15 à 60 ans, niveaux 0 à 3, 4 objectifs, 2 à 5 jours, 30 à 75 min, salle, maison ou dehors, priorités, douleurs, cartes). J'ai calculé le volume par muscle et par semaine en comptant 1 pour le muscle principal d'un exercice et 0,5 pour les muscles secondaires. Sorties brutes : `out.json`, `dump.txt`. Scripts : `run.js`, `nut.js`, `x.js` (même dossier).

**Ce qui est bon (à garder)** : découpages selon les jours (corps entier pour 2 et 3 jours, haut et bas pour 4 jours, 5 jours hybride), donc chaque muscle est travaillé 2 fois par semaine (Schoenfeld 2016). Repos de 150 s sur les gros exercices en prise de muscle (Singer 2024). Double progression avec ressenti Facile/Dur/Échec (Plotkin 2022). Semaine légère qui garde la charge et coupe le volume (Bell 2023). Reprise à −10 % après 14 jours. Moins de 18 ans limités à 8-15 répétitions, sans échec. Protéines de 1,6 à 2 g/kg (Morton 2018). Charges qui ne descendent jamais à 0.

---

## DANGEREUX

### D1. Un mineur qui choisit « Perdre du gras » reçoit un déficit calorique
Constat : une fille de 16 ans, 55 kg, 160 cm (IMC 21,5, poids normal) reçoit 1 650 kcal, soit −17 %. Un garçon de 15 ans (IMC 19) reçoit 1 900 kcal en déficit. Aucune protection dans `nutrition()`.
Risque : manque d'énergie pendant la croissance (RED-S), porte d'entrée vers les troubles alimentaires.
Correction dans `nutrition(p)` : si `+p.age<18`, alors `adj=0` quel que soit l'objectif (pas de surplus volontaire non plus). Sous le chiffre, afficher : « Moins de 18 ans : Stoa ne calcule pas de régime. Mange à ta faim, avec des protéines à chaque repas. Pour perdre du poids, parles-en à un médecin. » Dans le questionnaire, si l'âge est inférieur à 18, changer le libellé « Perdre du gras » en « Me sentir mieux dans mon corps » et le relier à l'objectif forme.
Source : Mountjoy 2018 (consensus CIO sur le RED-S) ; AAP, Golden 2016 (prévention de l'obésité et des troubles alimentaires chez l'adolescent : pas de régime).

### D2. Un déficit est proposé à une personne déjà maigre
Constat : une femme de 25 ans, 48 kg, 165 cm (IMC 17,6, maigreur) choisit « Perdre du gras » : 1 500 kcal (−17 %).
Correction : si l'IMC est inférieur à 18,5, griser « Perdre du gras » avec le texte : « Ton poids est déjà bas : perdre encore serait mauvais pour ta santé. Choisis plutôt Prendre du muscle ou Me remettre en forme. » Entre 18,5 et 20 : déficit limité à −10 %.
Source : OMS (seuils d'IMC) ; Mountjoy 2018.

### D3. Aucun plancher calorique
Constat : une femme de 60 ans, 45 kg, 150 cm, 2 séances par semaine : 1 100 kcal.
Correction : `kcal=Math.max(kcal, Math.round(bmr*1.1/50)*50, f?1200:1500)`. Plafonner aussi le déficit à −500 kcal par jour.
Source : NIH/NHLBI 1998 (pas de régime sous 1 200 kcal pour une femme ni sous 1 500 kcal pour un homme sans suivi médical) ; Helms 2014 (perte de 0,5 à 1 % du poids par semaine au maximum).

---

## FAUX

### F1. La durée annoncée n'est pas tenue
Constat : en choisissant 30 min, on obtient « 35 à 40 min » affichées (environ 45 min en réalité). En choisissant 45 min : 55 min affichées, environ 62 min en réalité. En choisissant 75 min avec la force au niveau 2 : 80 à 85 min affichées, environ 95 min en réalité. Trois causes :
- `sEst` ne compte ni le passage d'une machine à l'autre ni le réglage ;
- l'ajout des « petits muscles » (`NEED`) accepte `dur+8` ;
- la boucle de réduction tolère `dur+5`.

Correction :
- dans `sEst`, ajouter 75 s par exercice (`+ex.length*75`) et compter 45 s par série au lieu de 40 ;
- tolérance de `dur+3` partout, `NEED` compris (remplacer `dur+8` par `dur+3`) ;
- si on dépasse encore, retirer une série d'isolation avant d'ajouter quoi que ce soit.

Une séance plus longue que prévu est la première cause d'abandon.
Source : avis d'expert (temps mesurés sur le terrain).

### F2. Le volume n'est pas plafonné selon le niveau, et l'objectif « forme » reçoit plus de volume que l'objectif « muscle »
Constats :
- Débutante (niveau 0), 5 jours de 75 min, objectif muscle : **93 séries par semaine**, dont 21 pour les pectoraux et 23 pour le dos. C'est beaucoup trop pour une débutante : courbatures extrêmes, abandon, risque de rhabdomyolyse à l'effort.
- Niveau 3, objectif forme, 5 jours : **140 séries par semaine** (dos 29, fessiers 26). Profil de même niveau en objectif muscle : 105. Comme les repos sont plus courts, plus d'exercices tiennent dans le temps choisi.
- Niveau 2, objectif forme, 4 jours : quadriceps 14, ischios 13. Au même niveau en objectif muscle : 9 et 9. L'objectif « muscle » est le moins bien servi.

Correction : plafond de séries directes par muscle et par semaine, appliqué après la génération. On retire des séries d'isolation, puis des exercices, en commençant par le muscle le plus au-dessus du plafond.

| Niveau | muscle | gras | force | forme |
|---|---|---|---|---|
| 0 | 6-10 | 6-10 | 6-9 | 4-8 |
| 1 | 8-12 | 8-12 | 8-10 | 6-10 |
| 2 | 10-16 | 10-14 | 8-12 | 6-10 |
| 3 | 12-20 | 10-16 | 10-14 | 8-12 |

Ajouter aussi un plafond par séance : 8 séries directes au plus pour un même muscle (10 au niveau 3).
Source : Schoenfeld 2017 (dose-réponse) ; Pelland 2024 (rendements décroissants, plateau par séance autour de 6 à 11 séries) ; Baz-Valle 2022 ; ACSM 2009 (débutants : 1 à 3 séries par exercice).

### F3. Volume mal réparti : les pectoraux sont oubliés, les machines adducteurs et abducteurs sont gâchées
Constat : programme haut et bas sur 4 jours, niveaux 1 et 2, objectif muscle ou force : **pectoraux à 6 séries par semaine** contre 12 à 15 pour le dos. Au même moment, la séance « Bas 2 » contient abducteurs et adducteurs (6 séries). Même avec la priorité « pecs » (2 jours de 75 min) : seulement 7 séries.
Correction :
- dans `ul4`, mettre `fly` (ou `hpush2`) en 4e position de « Haut du corps 1 » ;
- dans « Bas du corps 2 », remplacer `add` par `calf` et ne garder `abd` que si « fessiers » est une priorité ;
- rapport poussée / tirage visé entre 1:1 et 1:1,3.

Une priorité doit **ajouter du volume**, pas seulement remonter l'exercice dans la séance : +3 à +4 séries par semaine pour le muscle prioritaire, dans la limite du tableau F2 + 4.
Source : avis d'expert ; Schoenfeld 2017.

### F4. Objectif « Perdre du gras » : répétitions plus hautes et repos plus courts, avec un message trompeur
Constat : 10-12 répétitions et 90 s sur les gros exercices, 12-15 répétitions et 60 s ailleurs, mode circuit à 45 s, et le texte « brûler un maximum… repos courts ». La musculation en déficit sert à **garder** le muscle : il faut garder des charges lourdes. Les repos courts dépensent quelques kcal de plus mais font perdre des répétitions, donc de la charge de travail.
Correction :
- dans `generate`, branche `gras` : mêmes fourchettes que muscle (`min=comp?6:10; max=comp?10:15`) ; repos `comp?120:75` ; garder le circuit en option (`min 60 s`) ;
- dans `sessionIntro`, remplacer le texte par : « Objectif : perdre du gras en gardant ton muscle. Garde des charges lourdes : c'est ce qui dit à ton corps de garder le muscle. Le gras, on le perd surtout avec l'assiette et les pas. »

Source : Helms 2014 (préparation en déficit) ; Murphy & Koehler 2022 (le déficit réduit la prise de muscle) ; Schoenfeld 2016 et Singer 2024 (repos courts : moins bons résultats).

### F5. Douleur au genou : le travail des quadriceps est presque supprimé
Constat : avec la carte « genou », les quadriceps tombent à 3 séries par semaine. Or le renforcement des quadriceps **est le traitement** de la douleur rotulienne et de l'arthrose du genou.
Correction :
- avec `sw.genou`, garder `quad2` et remplacer `quadIso` par `quad` en amplitude réduite ;
- remplacer le texte de la carte par : « Exercices de cuisses gardés, en amplitude confortable (presse, leg extension partielle, squat sur un banc). »
- ajouter la règle de la douleur : « Une gêne jusqu'à 3 sur 10 est acceptable si elle est revenue à la normale le lendemain ; au-delà, réduis la charge ou l'amplitude. »

Objectif : 6 à 9 séries de quadriceps par semaine.
Source : van der Heijden 2015 (Cochrane, douleur rotulienne) ; Bannuru 2019 (recommandations OARSI pour l'arthrose) ; Silbernagel 2007 (modèle de suivi de la douleur).

### F6. Objectif force : les gros mouvements ne sont faits qu'une fois par semaine, et les répétitions sont trop hautes pour les confirmés
Constat :
- programme haut et bas sur 4 jours, force, niveau 2 : squat barre 1 fois, soulevé de terre 1 fois et développé couché barre 1 fois par semaine (la 2e séance prend la machine) ;
- 5 à 8 répétitions sur tous les exercices de base, même pour un pratiquant de plus de 3 ans ;
- squat goblet en 4×5-8 aux niveaux 2 et 3 (n°43 et n°38) : c'est la tenue de l'haltère qui limite, pas les cuisses.

Correction :
- pour `goal==="force"` et un niveau ≥ 1, le 1er exercice de chaque séance est un des 3 mouvements (squat, développé couché, soulevé de terre ou leur version machine si dos fragile), chacun **2 fois par semaine** ;
- 1er exercice : niveau 0-1 en `4×5-6`, niveau 2-3 en `4×3-6`, repos 180-240 s, « 2 répétitions en réserve » ;
- 2e gros exercice : 6-10 ;
- isolation : 8-12 ;
- interdire `gobletSquat` dans le rôle de l'exercice principal en force (utiliser presse, hack squat ou squat barre).

Source : Schoenfeld 2017 et 2021 (charges lourdes pour la force) ; Grgic 2018 (fréquence et force) ; Kraemer & Ratamess 2004 / ACSM 2009 (1-6 répétitions, plus de 80 % du max pour les confirmés).

### F7. Le « Superman » est rangé dans « tirage horizontal (dos) »
Constat : chez soi ou dehors, il occupe la place du rowing et compte comme volume de dos. C'est un exercice des lombaires, pas des dorsaux ni des rhomboïdes. Au niveau 2 dehors, il remplace un vrai rowing.
Correction : `superman` en `pat:"core"`, `g:["Lombaires"]`. Pour le rôle `hpull` sans matériel, proposer un rowing à la serviette accrochée à une porte fermée (ou un rowing avec élastique). Si rien n'est possible, le dire dans l'intro : « Ton dos travaille peu sans matériel : un élastique à 10 € change tout. »
Source : avis d'expert (anatomie fonctionnelle).

### F8. Dehors au niveau 2 : 14 séries de quadriceps dans une même séance
Constat : séance « Bas 1 » : squat bulgare 4, pistol 4, squat au poids du corps 3, montées sur marche 3. « Haut 2 » : 11 séries de tractions. Le squat au poids du corps en 12-20 répétitions est trop loin de l'échec pour un pratiquant de niveau 2 et ne sert presque à rien.
Correction : plafond de séance de F2. Au niveau 2 et plus, exclure les exercices au poids du corps dont `HARDER` existe (`squatBW` → `bulgarian`). Ne pas choisir deux variantes de traction dans le rôle `bic` si `vpull` est déjà présent : utiliser `bagCurl` ou un curl avec élastique.
Source : Pelland 2024 ; Robinson 2024 (proximité de l'échec).

---

## AMÉLIORABLE

**A1. Ordre des exercices.** Un exercice de base est parfois placé après 2 ou 3 exercices d'isolation (n°1 A : tirage vertical en dernier, après les élévations latérales et les triceps ; n°9 C : développé couché machine après le curl ; n°6 C : développé couché barre après le développé machine). Correction : en corps entier, placer tous les exercices de base d'abord, en alternant bas, poussée et tirage, puis les isolations. Dans une même zone, l'exercice à la barre passe avant la machine.
Source : Simão 2012 ; Nunes 2021 (l'exercice fait en premier progresse le plus en force, peu d'effet sur le muscle).

**A2. Échauffement.** Le protocole actuel (50 % × 10, puis 75 % × 5) fatigue, avec 10 répétitions, et ne suffit pas en force. Correction :
- muscle, gras et forme : 50 % × 8 puis 75 % × 4 ;
- force, ou charge ≥ 60 kg : ajouter 90 % × 1-2 ;
- faire l'échauffement sur le **1er exercice de base de chaque zone** (bas, poussée, tirage), et non sur les 2 premiers exercices de la séance.

Source : avis d'expert ; Ribeiro 2020 (séries d'échauffement et performance).

**A3. « À l'échec » en haut de la fourchette.** Aujourd'hui, la charge ne monte pas. Atteindre le haut de la fourchette sur toutes les séries suffit à prouver qu'on est prêt. Correction : on monte d'une marche et on vise le bas de la fourchette, avec le texte « Monte, mais garde 1 répétition en réserve ». Garder la règle « consolider » seulement pour les mouvements à la barre libre, sans sécurité.
Source : Refalo 2023 (échec ou non : même gain de muscle) ; Helms 2016.

**A4. Grosses marches de charge sur l'isolation.** Élévations latérales de 2,5 à 5 kg, soit +100 %. Correction : si `e.inc/wk>0.2`, monter le haut de la fourchette à 20 répétitions avant d'augmenter la charge.
Source : Schoenfeld 2021 (de 6 à 30 répétitions, gains équivalents près de l'échec).

**A5. Semaine légère au calendrier.** Elle tombe même si la personne a sauté 1 ou 2 semaines. Correction : compter les **semaines réellement entraînées** (au moins 2 séances), et décaler la semaine légère de 1 semaine si 2 semaines ont été ratées dans le cycle. Débutants : toutes les 8 semaines plutôt que 6.
Source : Bell 2023 (consensus Delphi sur la semaine légère), avis d'expert.

**A6. Stagnation.** Après 3 séances sans progrès, l'app affiche seulement un message. Correction : proposer automatiquement −10 % sur la charge avec un bouton « Appliquer » ; après 2 cycles bloqués, proposer de changer d'exercice (alternatives déjà calculées).
Source : avis d'expert.

**A7. Plus de 55 ans.** Ajouter « monte vite, descends en 2-3 secondes » sur les exercices de base (puissance) et un exercice d'équilibre par séance (fente arrière tenue, marche en tandem 2 × 30 s).
Source : Fragala 2019 (position de la NSCA sur l'adulte âgé).

**A8. Débutant sur 4 ou 5 jours.** Au niveau 0, 5 jours donnent le découpage `ppl5`. Correction : niveau 0 et 5 jours → corps entier 3 jours + 2 jours de marche ou cardio, avec le texte « Commence par 3 séances : on ajoutera les autres dans 6 semaines ».
Source : ACSM 2009 (débutants : 2-3 jours, corps entier).

**A9. Dips sur chaise pour les débutants.** Ils mettent l'avant de l'épaule en extension extrême. Au niveau 0, préférer les pompes mains serrées inclinées.
Source : avis d'expert.

**A10. Protéines plafonnées à 160 g.** Un homme musclé de 100 kg (IMC 27, mais maigre) n'a alors que 1,6 g/kg. Correction : plafond à 200 g ; en déficit, fourchette de 1,8 à 2,2 g/kg du poids cible.
Source : Morton 2018 (limite haute 2,2 g/kg) ; Helms 2014.

---

## Question du propriétaire : permettre 2 objectifs ?

**Réponse : oui, au maximum 2, avec un objectif principal (premier choisi) et un objectif secondaire.** Le principal fixe les calories et les répétitions des gros exercices. Le secondaire ajoute un bloc limité. Il y a 6 paires possibles : 4 sont autorisées, 2 sont interdites, et l'objectif gras est bloqué dans certains cas.

### Paires autorisées

| Paire | Principal → dosage |
|---|---|
| **Muscle + Force** | **Muscle principal** : 1er exercice de chaque séance en 4×4-6 (repos 3 min) ; tout le reste comme muscle (6-10 sur les exercices de base, 10-15 en isolation) ; volume = tableau muscle ; calories +5 à +10 %. **Force principal** : 2 mouvements principaux par séance en 4×3-6 (niveau 0-1 : 5-6), repos 3-4 min ; puis 2-3 isolations en 8-12 × 2-3 séries ; volume = 80 % du tableau muscle ; calories +5 %. Pas de cardio imposé. |
| **Gras + Muscle** (autorisé si niveau 0-1 ou IMC ≥ 25 : la « recomposition » marche alors bien) | **Gras principal** : entraînement = muscle (6-10 / 10-15, repos de 120 s sur les exercices de base), volume = tableau gras ; calories −15 à −20 % ; protéines de 2,0 à 2,2 g/kg du poids cible ; 8 000 à 10 000 pas ; cardio léger 2 × 20 min, facultatif. **Muscle principal** : entraînement = muscle, volume plein ; calories à l'équilibre jusqu'à −10 % ; protéines 2,0 g/kg ; 7 000 à 8 000 pas. |
| **Gras + Force** | **Gras principal** : 1er exercice en 4×5-6, le reste comme muscle ; volume −20 % ; calories −15 à −20 % ; pas + cardio léger. **Force principal** : comme force, volume −20 % (le déficit fatigue) ; calories −10 % ; cardio à vélo plutôt qu'en course, au moins 6 h après la musculation ou un autre jour. |
| **Gras + Forme** | **Gras principal** : fourchettes 8-15, circuits possibles (repos ≥ 60 s) ; cardio 2 à 3 × 20-30 min, en pouvant parler (+ 1 séance de fractionné si niveau ≥ 1) ; calories −15 à −20 % ; 8 000 à 10 000 pas. **Forme principal** : comme forme ; 10 min de cardio en fin de séance ; calories −10 % ; 8 000 pas. |

### Paires interdites

| Paire | Phrase à afficher |
|---|---|
| **Forme + Muscle** | « Prendre du muscle te remet déjà en forme : choisis Prendre du muscle, et active le cardio si tu veux bouger plus. » |
| **Forme + Force** | « Devenir plus fort te remet déjà en forme : choisis Devenir plus fort, et active le cardio si tu veux bouger plus. » |
| **Gras + Muscle** au niveau 2-3 avec un IMC < 25 | « Après un an de muscu, perdre du gras et prendre du muscle en même temps devient très lent : fais d'abord l'un, puis l'autre. » |

### Blocages de l'objectif gras

| Cas | Phrase à afficher |
|---|---|
| **Gras + n'importe quoi** si âge < 18 | « Avant 18 ans, on ne fait pas de régime : choisis Me remettre en forme, Muscle ou Force. » |
| **Gras + n'importe quoi** si IMC < 18,5 | « Ton poids est déjà bas : perdre encore serait mauvais pour ta santé. » |

### Règles communes
- Le cardio sert la santé et la dépense ; il ne remplace jamais les séances de musculation. Repère : 150 à 300 min d'activité modérée par semaine, pas compris.
- Pas de troisième objectif.
- À l'écran : « Principal : X · En plus : Y ».

Sources : Barakat 2020 (recomposition, efficace surtout chez les débutants et les personnes en surpoids) ; Wilson 2012 (interférence entre cardio et musculation : course > vélo) ; Murphy & Koehler 2022 ; OMS 2020 (activité physique) ; Helms 2014 ; Schoenfeld 2017.

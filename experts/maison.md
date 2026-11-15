# Audit Stoa : programmes « maison » et « dehors »

Méthode : lecture de `LIB`, `HARDER`/`EASIER`, `candidates()`, `generate()`, `plan()` et du questionnaire. Avec Playwright : 24 profils générés (`profils.txt`), simulation de 8 à 30 séances par exercice avec `plan()` (`progression.txt`), test des boutons « ↑ version plus dure » (`swaps.txt`). Rien n'a été modifié dans l'app.

Bilan : la base est saine. Les exercices sont bien choisis pour les débutants (pompes inclinées, tractions négatives, rowing sous une barre basse), la plupart des consignes techniques sont bonnes et la progression par répétitions fonctionne pour les pompes. Le gros défaut est la chaîne des progressions : la version « plus dure » est proposée sans vérifier le matériel, les douleurs ni le niveau, et plusieurs maillons sont faux.

---

## DANGEREUX

### D1. Le bouton « ↑ Passer à » ignore les douleurs, le matériel et le niveau
- Preuve (`swaps.txt`) : une femme de 60 ans qui a déclaré une douleur aux genoux, sans chaise, se voit proposer squat → squat bulgare → **pistol squat**. Une personne dehors sans barres se voit proposer les **dips aux barres parallèles**. Le gainage mène au L-sit (niveau 2) même pour un débutant.
- Cause : `plan()` propose `HARDER[e.uid]` sans aucun filtre. Ensuite `generate()` applique `p.swaps` sans repasser par `candidates()` (`avoid`, `on`, `lvl`, `EXCL`).
- Correction : dans `plan()`, ne proposer `out.up` (et `out.down`) que si `candidates(LIBX[u].pat, PROG.p).some(x=>x.uid===u)`. Sinon, afficher seulement le texte `e.next`. Dans `generate()`, appliquer un swap seulement s'il passe le même filtre. Ajouter aussi un garde-fou d'âge : `jumpSquat` et `pistol` exclus dès 55 ans sans demande explicite.
- Source : avis d'expert. Recommandations ACSM pour les seniors et les douleurs articulaires : on adapte l'exercice à l'articulation, on ne force pas le passage (ACSM Guidelines, 11e éd.).

### D2. Les dips sur une chaise prescrits aux débutants et aux seniors, et servant de « version plus facile » des dips
- Preuve : on les trouve chez M1, M2, D1, D3, et chez D9 (femme de 60 ans, débutante). `EASIER.dips = chairDips`.
- Le problème : ce mouvement met l'épaule en extension et en rotation interne, chargée en fin d'amplitude. C'est l'un des exercices les plus souvent cités dans les douleurs d'épaule des pratiquants. Il n'est pas plus doux que les dips aux barres, donc c'est une mauvaise régression.
- Correction : mettre `chairDips` en `lvl:1` et l'exclure dès 55 ans. Pour les débutants, prendre à la place les pompes inclinées mains serrées ou l'extension triceps à l'élastique. Mettre `EASIER.dips` sur des dips négatifs (descente lente) ou des dips avec élastique, à ajouter dans `LIB`. Consigne : « garde les épaules en arrière et la poitrine ouverte, ne descends pas sous 90° ».
- Source : Kolber et al., *J Strength Cond Res* 2010 (traits du conflit d'épaule chez les pratiquants de musculation) ; avis d'expert.

### D3. Lest avec un sac à dos : consignes risquées
- `weightedPull` dit « Mets le sac sur le dos **ou devant toi** ». Devant, il gêne la traction et tire les bretelles sur le cou. `weightedDip` utilise aussi un sac sur le dos : la charge bascule vers l'arrière et met l'épaule en avant au point bas.
- Correction : « Sac porté haut et serré dans le dos, bretelles ajustées, 10 kg maximum. Au-delà, une ceinture de lest ou un haltère tenu entre les pieds. Vérifie les coutures et le contenu (pas de verre) avant chaque séance. » Même avertissement pour `bagRow` et `bagCurl` : sac solide, bretelles en bon état.
- Source : avis d'expert.

### D4. Matériel improvisé : il manque des consignes de sécurité
- Le squat bulgare et les dips sur chaise ne disent jamais « chaise calée contre un mur, sans roulettes ». Le rowing sous une table dit seulement « vérifie qu'elle est solide ». Il faut ajouter : « jamais de table pliante ou à tréteaux, mets-toi au milieu du grand côté, fais un premier essai genoux pliés ». La barre de porte (`negPull` et `pullup`) n'a aucune consigne de pose, et la chaise sert à monter en haut de la barre.
- Correction : ajouter une ligne de sécurité au premier point de `how` de chacun de ces exercices.
- Source : avis d'expert.

---

## FAUX

### F1. La boucle entre tractions négatives et tractions
- Preuve (`progression.txt`) : 8 négatives × 3 → passage aux tractions avec un minimum de 3 répétitions. La personne fait 3/2/1, l'app la renvoie aux négatives, puis lui repropose **tout de suite** les tractions, car elle relit l'ancienne séance de négatives réussie (s22→s23→s24). Ça tourne en rond, et c'est décourageant.
- Correction :
  1. Après un retour à la version plus facile, ne pas reproposer la version dure avant 2 nouvelles séances au maximum de la fourchette.
  2. Pour `pullup`, mettre `min:1` pendant les 4 premières semaines, en complétant chaque série par des négatives.
  3. Placer les tractions supination (`chinup`, plus faciles) avant les tractions en pronation.
  4. S'il y a des élastiques, ajouter des tractions assistées à l'élastique.
- Source : avis d'expert. C'est la progression classique en street workout : négatives → assistées → 1 répétition, puis on monte.

### F2. Retour à la version plus facile déclenché par une seule série ratée
- `under()` déclenche le retour si **n'importe quelle** série passe sous `min`. Exemple : pompes inclinées 9/8/7 puis 8/8/7 → « prends la version plus facile » dès la 3e séance, alors que la personne progresse. La baisse sur la dernière série est simplement de la fatigue normale.
- Correction : ne revenir en arrière que si la **1re série** est sous `min` deux fois de suite, ou si le total des répétitions baisse deux fois de suite.
- Source : avis d'expert. Une baisse de répétitions entre les séries est attendue avec 60 à 90 s de repos (Willardson 2006, *J Strength Cond Res*).

### F3. La charge des exercices lestés ne progresse jamais
- `weightedPull` et `weightedDip` sont en `unit:"corps"`. La branche poids du corps de `plan()` ignore les kg. Au maximum (8×3), le message répète sans fin « Ralentis la descente » (s6 à s10). Il ne dit jamais d'ajouter du lest.
- Correction : si `e.unit==="corps"` et que la charge notée est supérieure à 0, ou que l'exercice est `weightedPull`/`weightedDip`, utiliser la logique de charge : au maximum, +2,5 kg et retour à `min`.
- Source : surcharge progressive (ACSM Position Stand 2009, *Med Sci Sports Exerc*).

### F4. Les élastiques ne sont pas suivis
- L'app dit « note 0 » et « prends un élastique plus dur ». Au changement d'élastique, les répétitions chutent, ce qui déclenche « prends un élastique plus souple ». La boucle est la même qu'en F1, et la courbe de progrès devient fausse.
- Correction : noter le niveau d'élastique (1 à 5 ou une couleur) dans la case kg. Traiter un niveau plus haut comme une hausse de charge (repartir de `min` sans déclencher de retour).
- Source : avis d'expert.

### F5. Des « versions plus dures » qui changent d'exercice ou demandent du matériel absent
- `tableRow→invRow` : il n'y a pas de barre basse à la maison.
- `crunch→kneeRaise` : il faut une barre.
- `wallSit→jumpSquat` : on passe d'un exercice statique à un exercice sauté, ce qui ne convient pas en cas de douleur aux genoux.
- `gluteBridge→slideCurl` : on passe des fessiers aux ischios.
- `plank→lSit` : saut de deux niveaux.
- `EASIER.invRow = tableRow` : il n'y a pas de table dehors.
- Correction :
  - `gluteBridge→singleLegBridge` et `wallSit→bulgarian` ou `stepUp` plus haut (exercices à créer si besoin) ;
  - `plank→` gainage avec levée de bras, puis hollow body, puis L-sit genoux pliés sur deux chaises ;
  - `crunch→` relevés de jambes au sol ;
  - `tableRow→` pieds surélevés (le texte `next` existe déjà).
  - Associé à D1, cela suffit.
- Source : avis d'expert. Une progression doit garder le même mouvement et le même muscle.

### F6. L'objectif « Devenir plus fort » ne change rien sans charge
- Pour le poids du corps, `generate()` remplace les fourchettes par celles de `LIB` et fixe le repos à 90 s pour tous les objectifs. M12, D5 et D8 en « force » ont donc les mêmes squats 12-20 que les profils « forme ».
- Correction : si `goal==="force"`, prendre `max = min(e.max, 8)` sur les exercices de base, un repos de 150 à 180 s, et passer plus tôt à la version plus dure.
- Source : Schoenfeld et al. 2016, *J Strength Cond Res* (3 min de repos > 1 min pour la force) ; ACSM 2009.

### F7. Le leg curl avec une serviette est proposé dehors
- `slideCurl` est en `on:["rien"]`, il apparaît donc dehors (D2, D5, D6, D8). Or il faut un sol lisse : impossible sur l'herbe ou le béton.
- Correction : ajouter `not:["dehors"]`. Dehors, prendre le **Nordic curl** (pieds calés sous un banc ou une barre basse) ou le pont fessier sur une jambe, pieds sur le banc.
- Source : van der Horst et al. 2015 et Al Attar et al. 2017 (le Nordic curl réduit d'environ 50 % les blessures aux ischios).

### F8. Du matériel est prescrit sans que la personne l'ait confirmé
- La carte « table » n'apparaît que sans barre de traction. Pourtant M11, qui a une barre, reçoit le rowing sous une table. Les cartes au-delà de la 10e ne sont jamais posées : `meuble`, `table` et `sac` restent `undefined`, ce qui compte comme « oui ».
- Correction : sur ces trois cartes, traiter `undefined` comme `false` (`sw.table!==true`). Ou poser les cartes de matériel en premier, hors de la limite de 10.

### F9. Pas d'échauffement à la maison ni dehors
- `sEst` compte 5 minutes d'échauffement, mais l'app n'affiche aucun contenu, et `warm` n'est jamais vrai hors salle. On attaque donc les tractions, les dips ou le pistol à froid.
- Correction : un bloc fixe de 5 minutes en début de séance : rotations d'épaules, 20 jumping jacks ou marche sur place, 10 squats, 10 pompes inclinées faciles, 20 s de suspension à la barre si elle est utilisée.
- Source : Fradkin et al. 2010, *J Sci Med Sport* (l'échauffement améliore la performance) ; avis d'expert pour le contenu.

---

## AMÉLIORABLE

1. **Il manque un mouvement de charnière de hanche et le travail des ischios chez les débutants sans matériel.** M1, M2 et D1 ont 0 série pour les ischios, car `slideCurl` est de niveau 1. Ajouter le soulevé de terre sur une jambe au poids du corps (`hinge`, easy), le pont sur une jambe et les Nordic curls en négatif (niveau 2).
2. **Déséquilibre entre poussée et tirage.** M5 (16 ans, sans chaise, sans table, sans sac) : 10 séries de pectoraux, 0 vrai tirage (seulement le superman), 12 séries d'abdos. Il faut un tirage minimal sans rien : rowing avec une serviette autour d'une porte fermée, ou tirage à la porte en isométrie. Règle : tirage ≥ poussée en séries par semaine.
3. **Volume trop élevé chez les confirmés.** Triceps : 32 à 35 séries par semaine (M13, D8) ; fessiers : 34 à 38. La séance « Poussée » de D8 enchaîne 6 exercices de poussée, dont des dips sur chaise après des dips lestés. Plafonner à environ 20 séries difficiles par muscle et par semaine. Source : Schoenfeld, Ogborn & Krieger 2017 ; Pelland et al. 2024 (gains de plus en plus faibles).
4. **Volume trop bas chez certains seniors.** M3 (60 ans, 2 séances) et M15 (60 ans) : 3 séries de quadriceps et 3 de pectoraux par semaine. Pour lutter contre la fonte musculaire, viser au moins 6 séries par semaine pour les jambes. Source : ACSM seniors ; Borde et al. 2015, *Sports Med*.
5. **Exercices de niveau 2 sans test préalable.** « 1 à 3 ans de muscu » ne veut pas dire savoir faire 10 tractions ou un L-sit. Ajouter une question : « Combien de pompes et de tractions d'affilée ? ». Le niveau des exercices au poids du corps en dépendrait.
6. **L-sit « au sol ».** `lSit` est en `on:["rien"]`, et l'app le prescrit 3 fois par semaine (M14, D8) sans barres. C'est presque infaisable au sol. Le mettre sur deux chaises ou des parallettes, et commencer par la version genoux pliés.
7. **Dehors, « barres = oui » suppose barre fixe + barres parallèles + barre basse.** Demander lesquelles sont présentes.
8. **Squats sautés pour l'objectif « muscle ».** M9 et M12 en ont dans l'emplacement d'isolation des cuisses. C'est peu efficace pour l'hypertrophie, et l'impact est fort pour les personnes lourdes (M11 pèse 85 kg). Préférer le squat bulgare en tempo lent ou les montées sur une marche haute ; garder les sauts pour « gras » et les moins de 45 ans.
9. **Haltères légers (5 kg).** M8 reçoit un squat goblet et un soulevé de terre jambes tendues à 5 kg, ce qui revient presque au poids du corps. Avec `dbMax` ≤ 8, préférer les versions sur une jambe (squat bulgare et soulevé de terre sur une jambe avec haltères).
10. **Cardio inadapté au lieu.** L'intro dit « vélo, rameur… » à la maison et dehors. Proposer plutôt : marche rapide, montées d'escaliers, corde à sauter, intervalles de jumping jacks. Pour l'objectif « forme », proposer un peu de cardio par défaut (OMS : 150 à 300 min d'activité modérée par semaine).
11. **Pompes piquées.** Remplacer « le haut de la tête vers le sol » par « le front devant les mains (triangle tête-mains), coudes à 45° ».
12. **Rowing avec un sac à dos et mal de dos.** `bagRow`, buste penché, sort chez D8 qui a une douleur au dos. Ajouter `avoid:["dos"]`, ou un appui de la main libre sur une chaise.
13. **Le choix « banc » ne sert à rien à la maison** : aucun exercice n'utilise ce matériel. Utiliser le banc pour le développé aux haltères incliné, les dips sur banc, le Nordic curl et le hip thrust.

---

Fichiers produits : `profils.txt` (24 profils), `progression.txt` (simulations), `swaps.txt` (test des versions plus dures) et les scripts `gen.js`, `sim.js`, `swap.js`, tous dans ce dossier.

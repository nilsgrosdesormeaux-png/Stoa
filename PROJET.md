# Stoa : document de reprise du projet

Ce document résume tout ce qui a été fait et décidé, pour reprendre le projet depuis un autre compte Claude sans rien perdre.
Dernière mise à jour : 14 novembre 2026 (questionnaire et générateur de programmes ajoutés).

---

## 1. Ce qu'est Stoa aujourd'hui

Une web app de musculation, installable sur iPhone comme une vraie application (PWA) : icône sur l'écran d'accueil, plein écran, fonctionne sans réseau à la salle.

- **Fichiers** : tout tient dans `index.html` (HTML, CSS et JS dans un seul fichier), plus `sw.js` (fonctionnement hors ligne), `manifest.webmanifest` (installation), les icônes `icon-*.png` et le dossier `photos/` (photos des vraies machines de la salle).
- **Hébergement** : GitHub Pages sur ce dépôt (branche `main`, dossier racine). Adresse attendue : `https://nilsgrosdesormeaux-png.github.io/Stoa/`. À vérifier : Settings, puis Pages, doit être activé sur `main`.
- **Installation sur iPhone** : ouvrir l'adresse dans Safari, puis Partager, puis « Sur l'écran d'accueil ».
- **Version Claude** : la même app existe en artefact sur le compte Claude de Nils. Elle ne se met à jour que depuis ce compte. Les données y sont synchronisées entre appareils.
- **Données** : stockées uniquement sur l'appareil (localStorage, clé `stoa-v1` ; photos dans IndexedDB `stoa-photos`). Rien n'est envoyé sur internet. Exporter et Importer (onglet Progrès) permettent de passer d'un appareil à l'autre.
- **Le dépôt est public** : le code, les photos de machines et le programme sont visibles par tous, pas les données personnelles.

### Pour tout le monde depuis le 14 novembre 2026 (questionnaire + générateur)
Stoa n'est plus seulement l'app de Nils : n'importe qui peut ouvrir le lien et obtenir son propre programme.
- **Au premier lancement**, un questionnaire plein écran (2 minutes) : prénom, où t'entraîner (salle avec choix de la chaîne / chez moi avec le matériel / dehors avec ou sans barres), objectif (perdre du gras, prendre du muscle, remise en forme, force), sexe, âge, taille, poids, niveau, séances par semaine (2 à 5) et durée (30 à 75 min), 2 priorités (fessiers, jambes, ventre, dos, épaules, bras, pecs), douleurs (genoux, bas du dos, épaules), puis **des cartes à glisser façon Tinder** (machines rassurantes ? peur de mal faire ? cardio ? abdos à chaque séance ? charges lourdes ? varier les exercices ?). Écran final : résumé du programme, calories et protéines, case « Stoa ne remplace pas un médecin » à cocher.
- **Le générateur** (fonction `generate()` dans `index.html`) :
  - le nombre de jours choisit le découpage : 2 = corps entier A/B, 3 = corps entier A/B/C (ou Haut/Bas/Corps entier si plus d'un an de pratique), 4 = Haut/Bas ×2, 5 = Haut/Bas/Poussée/Tirage/Jambes ;
  - la durée fixe le nombre d'exercices (4 à 7) ;
  - l'objectif fixe séries, répétitions et repos ; l'âge (moins de 16 ans, plus de 55 ans) les adapte ;
  - les exercices viennent d'une **bibliothèque** (`LIB`, environ 60 exercices : machines, haltères, élastiques, poids du corps, barres), filtrée par le lieu, le matériel, le niveau et les douleurs ;
  - les priorités passent en début de séance ; les autres options d'un même rôle deviennent les « alternatives » ;
  - charges de départ estimées selon le sexe, le niveau et le poids (toujours « à ajuster ») ;
  - calories (formule de Mifflin-St Jeor) et protéines calculées pour le Guide.
- **Exercices au poids du corps** : on progresse en répétitions (ou en secondes pour le gainage), puis l'app propose la version plus dure. Records de répétitions ajoutés.
- **Mode « première séance »** dans l'onglet Séance pour rassurer (arriver à la salle, trouver une machine, commencer léger).
- **Nouvelle partie « Douleur : quand s'arrêter »** dans le Guide, pour tout le monde.
- **Ton programme (Nils) est intact** : un téléphone qui a déjà des séances enregistrées garde automatiquement « le programme de Nils » (profil `mode: "nils"`). Onglet Programme : « Créer mon programme (questionnaire) », et retour possible au programme de Nils.
- Les **photos des machines On Air** ne s'affichent que pour quelqu'un qui a choisi On Air (les machines changent d'une salle à l'autre).
- Nouveaux dessins simples pour les exercices au poids du corps et aux haltères (pompes, squat, tractions, fentes, pont fessier…). Ils restent à améliorer.

### Ajouts du 14 novembre 2026 (suite)
- **Points forts / points faibles** : dans le questionnaire, une carte par partie du corps avec une petite silhouette (zone en rouge). À droite = à travailler en priorité (étoile ★ dans « Séries par semaine et par muscle »), à gauche = point fort (une série de moins, entretien), au milieu = normal.
- **Le tableau « Séries par semaine et par muscle » avec les étoiles est gardé** : Nils y tient.
- **Exercices animés** : un personnage stylisé fait le mouvement en boucle, muscle travaillé en rouge (moteur `ANIM` dans `index.html`, environ 35 animations). Ils remplacent les dessins fixes et les photos de Nils (jugées pas assez belles).
- **Photos des machines** : les photos libres trouvées sur Wikimedia ont été écartées (amateures, avec des gens, styles différents). Décision : générer des images par IA, **toutes dans le même style**, sans personne, sans logo. Les consignes sont dans `PROMPTS-IMAGES.md`. Il faut connecter un outil de génération (Adobe Firefly déjà installé dans Claude, à autoriser) ou les générer soi-même.
- **Images des machines générées** (14 nov.) : modèle MAI Image 2.6 via Pollinations (compte gratuit relié au GitHub de Nils, clé enregistrée dans l'environnement Claude). Style unique : machine vide, fond noir, reflet rouge, sans logo. Prêtes : presse, leg extension, leg curl, hip thrust, abducteurs. À finir quand le crédit gratuit se recharge : le reste de la liste de `PROMPTS-IMAGES.md` (le développé assis est à refaire). Ajouter chaque nouvelle image dans `MREADY` (index.html) et dans `CORE` (sw.js).
- **Vacances / pause** (onglet Programme) : programme temporaire pour X semaines (chez soi, dehors, autre salle, avec le matériel sur place), puis retour automatique au vrai programme. Pause totale possible. Explication de ce qu'on perd selon la durée (rien jusqu'à 1 semaine, presque rien jusqu'à 3, petite perte vite récupérée ensuite). La flamme est protégée pendant la pause. Au retour après 2 semaines ou plus : charges -10 % sur la première séance.
- **Niveau plus précis** : 4 niveaux (jamais, moins d'un an, 1 à 3 ans, plus de 3 ans) et, à la salle, les charges actuelles (presse, développé couché, tirage vertical) pour démarrer directement au bon poids.
- **Tests à grande échelle (14 nov.)** : 1000 utilisateurs simulés (10 semaines chacun, ~33 000 séances) + 4 équipes d'agents testeurs (débutants salle, confirmés salle, maison/dehors, experts : coach, Nils, directeur artistique). Rapports dans la session Claude. Corrigés : charges des haltères plafonnées au matériel, 0 répétition possible, boutons « version plus dure / plus facile » et « Changer d'exercice » (retenus pour la suite), minimum de séries pour bras et épaules, durée respectée (séances complétées ou raccourcies), points forts moins réduits, abdos ajoutés sans retirer d'exercice, exercices à la barre libre pour confirmés/force, exercices avancés de street workout, rowing et curl au sac à dos (parc sans barres), douleurs mieux filtrées (tractions et épaules, leg extension et genoux, superman et dos), historique lié au programme (plus de séance « déjà faite » après vacances), baisse de 10 % seulement après une vraie absence, pause arrêtable, numéros d'urgence (15/112), textes adaptés à la maison, nouvelles animations.
- **Reste à faire (rapports)** : écran Séance trop chargé en texte, contrastes du thème clair, records et RM estimé pour confirmés, export CSV, choisir ses jours, finir les images de machines.
- **Mode test** : ajouter `#test` à la fin de l'adresse donne une app vierge, séparée des vraies données (pratique pour tester le questionnaire sur son propre téléphone ou son ordinateur).

### Correction du 14 novembre 2026 (soir) : le programme de Nils
- Nils tient à son programme d'origine et à **ses photos des machines On Air** : elles s'affichent de nouveau dans son programme (`USE_GYM_PHOTOS` vrai en mode Nils ; les animations restent pour les exercices sans photo et pour les programmes générés).
- Premier écran du questionnaire : bouton « Reprendre le programme de Nils » (utile sur une nouvelle installation sans données).
- Onglet Programme : « Revenir au programme de Nils » est toujours visible quand un autre programme est actif.
- Ne plus retirer ses photos ni modifier son programme sans lui demander.

### Les écrans (barre d'onglets en bas, style iOS)
1. **Accueil** (tableau de bord « Hammer Night », noir et rouge) :
   - anneau des séances de la semaine (x/3) et série de semaines complètes d'affilée (flamme façon Snap) ;
   - calendrier de la semaine et phrase de motivation selon la situation ;
   - prochaine séance avec un gros bouton et la charge à viser ;
   - charge soulevée dans la semaine (avec l'équivalent en « Clio »), totaux et records récents ;
   - courbes de 3 exercices clés, séries faites par muscle cette semaine, poids avec la zone objectif 70-71 kg ;
   - prénom réglable en touchant le rond en haut à droite.
2. **Séance** : sélecteur A, B, C, Bonus.
   - Une carte par exercice : vignette photo ou dessin, séries × répétitions, repos, muscle, réglages de machine notés par l'utilisateur, conseil de progression.
   - Saisie série par série (valeurs proposées en gris), séries d'échauffement « É » qui ne comptent pas dans les progrès, ressenti après chaque exercice (Facile / Dur / À l'échec, décochable).
   - Chrono de repos qui se lance tout seul et devient vert à la fin ; on le touche pour changer la durée, retenue par exercice.
   - Records en direct, bilan de fin de séance, confirmation si on termine avant la fin.
   - Fiche « Machine et consignes » : photo, comment reconnaître la machine, comment faire, pourquoi, temps de repos, charge à noter, alternatives si la machine est prise, bouton pour ajouter sa propre photo (galerie ou appareil).
3. **Programme** : semaine type, cycle de 6 semaines, explication des temps de repos, séries par muscle et par semaine, règles.
4. **Progrès** :
   - poids (moyenne sur 7 jours, tendance si au moins 3 pesées) ;
   - photos de suivi (stockées uniquement sur le téléphone, première et dernière côte à côte) ;
   - mensurations, facultatives ;
   - liste des charges par exercice avec historique et courbe ;
   - sauvegarde : exporter ou importer.
5. **Guide** : vocabulaire, déroulé d'une séance, règles de progression, plan adapté au physique, nutrition, photos de suivi, erreurs classiques, installation sur iPhone.

### Identité
- **Nom** : Stoa (le portique où enseignaient les stoïciens : discipline, volonté, maîtrise de soi ; un nom évocateur, pas descriptif, comme « On Air »).
- **Logo** : un S qui monte vers un sommet, S ivoire et sommet rouge sur fond noir (`icon-512.png`, repris d'une maquette choisie par Nils). Il est un peu flou en très grand : à redessiner proprement en vectoriel.
- **Couleurs « Hammer Night »** : fond `#0A0A0C`, rouge `#E8132B` (et `#FF4757`), or `#FFB224` pour les records, vert `#32D74B` pour les hausses. Typographie système iOS, chiffres en très gras. Un thème clair existe aussi.

---

## 2. L'utilisateur actuel : Nils (pour qui le programme est fait)

- Homme, 20 ans, environ 70 kg, 1,79 m. Débutant, environ 2 mois de pratique.
- Salle **On Air Angers Saint-Serge** : beaucoup de machines Hammer Strength à disques (rouges), plus Technogym, Life Fitness, Panatta, Gym 80, Eleiko.
- **Points forts** : pectoraux, taille fine de face.
- **Points faibles** : largeur des épaules (deltoïdes latéraux), largeur et épaisseur du dos, bras (surtout triceps), épaules enroulées vers l'avant. Un peu de gras sur le bas du ventre et les flancs.
- **Stratégie : recomposition corporelle**, pas de prise de masse pour l'instant :
  - 2 500 à 2 600 kcal pour stabiliser le poids (il avait pris 3 kg en un mois) ;
  - protéines 125 à 140 g par jour, créatine 3 à 5 g par jour, sommeil 7 à 9 h ;
  - photos de suivi une fois par mois.
- **Il tient à garder SES exercices**, ceux qu'il fait à la salle. Ne pas les remplacer sans lui demander.
- Il ne maîtrise pas le vocabulaire technique : tout expliquer simplement.

### Programme actuel (3 séances + 1 Bonus)
Planning : lundi A, mercredi B, vendredi C, dimanche Bonus (48 h après C). Ne jamais faire A et C deux jours de suite.

| Séance | Exercices (séries × répétitions, repos, charge de départ) |
|---|---|
| **A** (pecs du haut, dos largeur, épaules, triceps) | Développé incliné Hammer 3×6-10, 2 min 30, 25 kg · Tirage vers le bas prise large 4×8-12, 2 min 30, 35 kg · Développé à la Smith 3×8-10, 2 min, 20 kg · Élévations latérales poulie 3×12-20, 1 min, 2,5 kg · Extension triceps poulie haute 3×10-15, 1 min 30, 26 kg · Extension triceps au-dessus de la tête, poulie basse 3×10-15, 1 min 30, 12 kg |
| **B** (quadriceps, ischios, fessiers, adducteurs, abdos) | Presse à cuisses 4×8-12, 2 min 30, 80 kg (40 par côté) · Leg curl Hammer 4×10-12, 1 min 30, 30 kg (15 par côté) · Leg extension 3×12-15, 1 min 30, 65 kg · Abducteurs 2×15-20, 1 min, 95 kg · Adducteurs 2×15-20, 1 min, 95 kg · Crunch à la poulie haute 3×10-15, 1 min, 20 kg · Mollets à la presse (facultatif) |
| **C** (dos épaisseur, arrière d'épaules, épaules, pecs, biceps) | Rowing Hammer prise neutre 4×8-12, 2 min 30, 25 kg · Tirage vertical prise neutre 3×10-12, 2 min, 35 kg · Développé couché Hammer allongé 3×8-12, 2 min, 20 kg · Élévations latérales 4×12-20, 1 min · Tirage arrière d'épaule à la poulie, un bras 3×12-20, 1 min 15, 3 kg · Curl pupitre Hammer 3×8-12, 1 min 30, 25 kg |
| **Bonus** (biceps, triceps, épaules, bas des pecs) | Curl machine 3×10-12, 50 kg · Triceps poulie haute 3×10-15 · Élévations latérales 3×12-20 · Arrière d'épaule 3×15-20 · Développé assis bas des pecs 3×10-12 · Développé épaules (facultatif) · Crunch (facultatif) |

Charges notées : sur une machine à disques, le **total des disques** (gauche + droite) sans le poids de la machine, affiché avec « (X par côté) » ; sur une machine à pile, le chiffre de la pile.

### Logique de progression (fonction `plan()` dans `index.html`)
- **Double progression** : +1 répétition par série jusqu'au haut de la fourchette sur toutes les séries, puis +1 marche de charge (2,5 kg en général, 5 kg pour la presse et le leg curl) et retour au bas de la fourchette.
- **Ressenti « Facile »** : +2 répétitions par série, ou une double marche si on était déjà en haut de la fourchette ; deux fois « Facile » à la même charge = hausse.
- **Ressenti « Dur »** (la cible) : règle normale. Sans réponse, l'app considère « Dur ».
- **Ressenti « À l'échec »** : mêmes répétitions ; en haut de la fourchette, on consolide sans monter. Deux échecs de suite sans progrès = −1 marche.
- **Deux séances sous le bas de la fourchette** : −1 marche.
- **Stagnation** (3 séances sans progrès) : message conseil.
- **Cycle de 6 semaines** aligné sur le lundi : la 6e est une semaine légère (moitié des séries, 3 répétitions en réserve), exclue des calculs de progression.
- **Échauffement** proposé (50 % × 10, puis 75 % × 5) sur les premiers exercices.

### Sources scientifiques citées par l'agent coach (non relues une à une)
- **Repos** : Schoenfeld 2016, Grgic 2017, Singer 2024.
- **Volume** : Schoenfeld 2017, Pelland 2024.
- **Fréquence** : Schoenfeld 2016 et 2019.
- **Charges lourdes ou légères** : Schoenfeld 2017.
- **Proximité de l'échec** : Refalo 2023, Robinson 2024.
- **Double progression** : Plotkin 2022.
- **Exercices précis** : triceps au-dessus de la tête, Maeo 2023 ; leg curl assis, Maeo 2021.
- **Protéines** : Morton 2018.

Les valeurs exactes (2 min 30, semaine légère toutes les 6 semaines, échauffement) relèvent de l'avis d'expert.

---

## 3. Ce qui a été fait pour arriver là

1. Première version reprise d'un ancien artefact Claude, puis reconstruction complète.
2. **Audit d'un agent coach sportif** : programme, repos, progression, textes. 4 bugs de progression corrigés, dont des séries vides qui faisaient disparaître des séances.
3. **Audit d'un expert UX mobile** : barre d'onglets en bas, cartes plus courtes, saisie avec virgule, contrastes, chrono qui ne disparaît plus, confirmation en fin de séance.
4. **Tests par 4 profils** : débutant, intermédiaire habitué à Hevy et Strong, coach pro, directeur artistique. L'écran d'accueil vient de la maquette du directeur artistique. Le rapport de l'intermédiaire n'a jamais été rendu.
5. **Photos des vraies machines intégrées** :
   - développé incliné, développé couché et Smith Hammer ;
   - rowing Hammer et curl pupitre Hammer ;
   - presse à cuisses Hammer et leg curl Hammer ;
   - leg extension.
6. Nom, logo et dépôt séparé du projet A-point (un autre projet de Nils, sans rapport).

### Retours des testeurs pas encore traités
- **Débutant** :
  - un mode « un exercice à la fois » ;
  - moins de texte entre deux séries ;
  - une vibration en fin de repos (impossible sur iPhone en web, d'où le signal visuel) ;
  - des mots encore flous : « Hammer », « ischios », cycle et semaine affichés en même temps.
- **Coach** :
  - jambes travaillées une seule fois par semaine (ischios un peu faibles) ;
  - aucune règle sur la douleur : courbature normale, douleur vive = arrêter et prendre l'alternative, plus de 7 jours = kiné ;
  - pas de question sur le sommeil avant la séance ;
  - pas de records en répétitions, seulement en charge ;
  - choisir l'alternative quand la machine est prise, avec son propre historique.

---

## 4. Reste à faire tout de suite
- [ ] **Faire tester le questionnaire** à la copine de Nils et à 2-3 potes (salle, maison, dehors) et noter ce qui bloque.
- [ ] Faire relire la bibliothèque d'exercices et les règles du générateur par un agent coach.
- [ ] Listes de machines par chaîne (Basic-Fit, Fitness Park…) : pour l'instant, toutes les salles sont supposées avoir le matériel standard.
- [ ] Choisir ses jours d'entraînement (aujourd'hui : jours proposés automatiquement).
- [ ] Le côté jeu : niveaux, badges, défis entre amis.
- [ ] Activer GitHub Pages (Settings, puis Pages, Branch `main`, dossier root) et tester l'installation sur iPhone.
- [ ] Ajouter la photo de la machine Life Fitness « Hip Abduction / Adduction » (la même pour les deux exercices), dans `photos/` et dans la table `PHOTO` de `index.html` (uid `abductor` et `adductor`), plus dans la liste `CORE` de `sw.js`.
- [ ] Redessiner le logo en vectoriel (même dessin : S ivoire qui monte vers un sommet rouge).
- [ ] Améliorer les illustrations des exercices sans photo (les dessins actuels sont jugés trop basiques).
- [ ] Après chaque modification : changer `CACHE` dans `sw.js` (dernière valeur : `stoa-v3`) pour que les iPhone récupèrent la nouvelle version.

---

## 5. La vision : une vraie application, un « coach de poche »

**Objectif de Nils** : remplacer un coach sportif (environ 60 € de l'heure) pour les débutants qui ont peur d'aller à la salle, ne connaissent pas les machines et manquent de motivation. Exemple type : sa copine.

### Idées retenues (rien n'est encore construit)
- **Questionnaire d'inscription très simple, façon Tinder** : des cartes à glisser à gauche ou à droite (« machines ou poids libres ? », « séances courtes et intenses ou longues ? »…), plus quelques champs : sexe, âge, taille, poids, objectif, jours disponibles, niveau, douleurs éventuelles.
- **Pas seulement pour la salle** : première question « Où veux-tu t'entraîner ? » avec trois choix :
  - **salle** : choix de la chaîne (Basic-Fit, Fitness Park, L'Orange Bleue, On Air…) puis des machines du club ;
  - **dehors** : street workout, barres dans un parc ;
  - **chez moi** : matériel disponible (rien, élastiques, haltères, barre de traction, banc…).
- **Générateur de programmes plutôt que 100 programmes écrits à la main** :
  - les jours disponibles décident du découpage (2 jours = corps entier, 3 = A/B/C, 4 = haut/bas, 5-6 = poussée/tirage/jambes) ;
  - l'objectif (perdre du gras, prendre du muscle, force, remise en forme) décide des répétitions, du volume et des repos ;
  - le profil ajuste ;
  - le lieu et le matériel choisissent les exercices dans une bibliothèque où chaque mouvement a plusieurs variantes.
- **Base des salles** : une liste de machines par chaîne, puis chaque utilisateur coche ou photographie les machines de son club, et ça profite à tous. Citer les noms des salles est possible, pas leurs logos sans accord.
- **Côté jeu** :
  - flamme de régularité façon Snap (déjà en place), niveaux et expérience, badges ;
  - défis de la semaine, défis entre amis ou en couple ;
  - toujours au service de la régularité : jamais de culpabilisation, jamais d'incitation à se blesser.
- **Démonstrations de qualité** :
  - animations 3D d'exercices sous licence (banques payantes utilisées par la plupart des apps) ;
  - ou courtes vidéos tournées soi-même sur fond neutre ;
  - ou un illustrateur pour un style unique ;
  - plus tard : correction de posture par la caméra.
- **Mode « première visite »** pour rassurer : où aller, comment régler une machine, quoi faire en arrivant.

### Pour en faire une vraie application
- Comptes utilisateurs et base de données en ligne (Supabase, déjà utilisé par Nils sur un autre projet).
- Publication sur l'App Store : reconstruire en application native ou en React Native/Expo, en réutilisant la logique et le design. Compte développeur Apple : 99 € par an.
- RGPD : poids, mensurations et photos sont des données sensibles (consentement, suppression possible, hébergement en Europe). Avertissement santé obligatoire.
- **Concurrence à étudier** : Hevy, Strong, Fitbod, Alpha Progression, l'app de Basic-Fit et les apps de fitness à la maison. Angle différenciant pressenti : débutants, français, simple et beau, adapté à TA salle et TES machines (ou à ta maison), programme créé en 2 minutes.

### Ordre conseillé
1. **Maintenant** : Nils utilise Stoa lui-même 4 à 8 semaines. C'est le meilleur test.
2. **Étude de marché** : concurrents, prix, ce que les utilisateurs leur reprochent dans les avis, où Stoa peut faire mieux.
3. **Document de conception** : parcours d'une débutante du premier écran à sa première séance, questionnaire en cartes, règles du générateur, système de jeu.
4. **Prototype web** : questionnaire, générateur, 2 ou 3 chaînes de salles et le mode maison. Le faire tester à 10 à 20 personnes.
5. **Ensuite** : comptes en ligne, plus de salles, passage sur l'App Store si les testeurs accrochent.

---

## 6. Façon de travailler souhaitée par Nils
- Expliquer simplement, sans jargon. Il parle souvent à l'oral, de façon informelle.
- Il aime qu'on fasse tester par des agents spécialisés (coach, débutant, designer, commercial) et qu'on lui donne un avis franc.
- Il veut un rendu « stylé », premium, qui donne envie.
- Ne pas mélanger avec le projet A-point.
- Ne pas changer ses exercices sans lui demander.

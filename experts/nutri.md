# Audit nutrition et poids — Stoa

Périmètre lu : `nutrition(p)` (l. 2043), `bwNote()` (l. 2118), carte poids de l'accueil (l. 2148-2158), onglet Progrès (l. 2779-2793), Guide `guideGen()` (l. 2859-2894) et Guide Nils (l. 2963-2986), fin de séance (l. 2601), questionnaire « body » (l. 3493-3558), écran final (l. 3513-3518).
Tests : 28 profils avec Playwright (`nutrition` + `generate`), résultats dans `res.tsv`, script `t.js`, dans le même dossier que ce rapport.

## Ce qui est bon (à garder)

- Mifflin-St Jeor : c'est la meilleure formule pour les adultes, y compris en surpoids (Frankenfield 2005, ADA).
- Facteur d'activité 1,30 + 0,06 × séances (1,42 à 1,60) : réaliste pour quelqu'un qui travaille assis. Tous les résultats restent au-dessus du métabolisme de base (au plus bas 1,16 × BMR).
- Déficit de 17 % (« perdre du gras ») : modéré, dans la zone des recommandations (ISSN 2017 « diets and body composition » : 500 kcal environ, 0,5 à 1 % du poids par semaine).
- Protéines 1,6 à 2,0 g/kg, en 3-4 repas : conforme (Morton 2018 ; ISSN 2017 « protein and exercise » : 1,4 à 2,0 g/kg, 0,25 à 0,4 g/kg par repas).
- Protéines calculées sur un poids « IMC 25 » en surpoids : bonne idée (évite 240 g/j chez une personne de 120 kg).
- Équivalences alimentaires : toutes justes à ± 10 % (CIQUAL ANSES) : poulet cru 150 g ≈ 34-35 g ; 3 œufs ≈ 19-20 g ; skyr 150 g ≈ 15-16 g ; thon égoutté 100 g ≈ 25 g ; lentilles cuites 100 g ≈ 9 g ; tofu 150 g ≈ 18-22 g ; lait 500 ml ≈ 16-17 g. Repas post-séance : 2 skyrs ≈ 31 g, lait + avoine + skyr ≈ 39 g : justes.
- Créatine (texte Nils) : 3 à 5 g, sans phase de charge, tous les jours, eau 0,5-1 kg : conforme (ISSN 2017 « creatine »).
- Eau 1,5 à 2,5 L de boissons : cohérent avec EFSA 2010 (2,0 L F / 2,5 L H au total, aliments compris).
- Recomposition pour Nils (20 ans, 70 kg, 1,79 m) : 2 500-2 600 kcal ≈ son maintien calculé (≈ 2 550), 125-140 g = 1,8-2,0 g/kg : juste.

## DANGEREUX

### D1. Déficit calorique appliqué aux mineurs (12 à 17 ans)
Constat : le questionnaire accepte dès 12 ans. Une fille de 14 ans, 45 kg, objectif « perdre du gras » reçoit 1 600 kcal (−17 %) ; un garçon de 16 ans, 85 kg, 2 250 kcal. Le Guide dit pourtant « mange à ta faim, pas de régime strict » : contradiction. Mifflin n'est pas validé avant 18 ans et ignore la croissance. L'adolescence est la période à plus haut risque de troubles du comportement alimentaire (TCA).
Correction :
```js
if(a<18){ // adolescents : jamais de déficit, pas de chiffre de calories
  adj = 0;  // et pas de surplus chiffré non plus
  return {kcal:null, prot:[Math.round(w*1.2/5)*5, Math.round(w*1.6/5)*5], bmi, teen:true};
}
```
Affichage à la place des calories : « **Mange à ta faim**, 3 repas et 1 à 2 collations. Ton corps grandit : ce n'est pas le moment de faire un régime. » Pour l'objectif « perdre du gras » chez un mineur, remplacer le texte du Guide par : « À ton âge, on ne fait pas de régime. On bouge plus, on dort bien, on limite les boissons sucrées et le grignotage devant les écrans. Si ton poids t'inquiète, parles-en à ton médecin. »
Protéines ado : 1,2 à 1,6 g/kg suffisent (avis d'expert ; besoins ANSES ados ≈ 0,9 g/kg, sportif ×1,5).
Sources : ANSES 2016 (références nutritionnelles), AAP 2016 « Preventing obesity and eating disorders in adolescents », HAS 2019 (obésité de l'enfant).

### D2. Objectif « perdre du gras » possible en maigreur
Constat : femme 22 ans, 165 cm, 45 kg (IMC 16,5) → 1 550 kcal de déficit, sans aucune alerte. C'est le profil type d'une anorexie.
Correction (adultes) :
- IMC < 18,5 : objectif « perdre du gras » **bloqué**. Message : « Ton poids est déjà bas pour ta taille. Perdre encore n'est pas bon pour ta santé : on part sur "me remettre en forme" ou "prendre du muscle". Si tu te trouves trop gros(se) malgré tout, parles-en à un médecin, c'est important. » `adj = 0` (ou +150 si muscle).
- IMC 18,5 à 20 + « perdre du gras » : déficit limité à 10 % (`adj=-0.10*tdee`) et texte « tu as peu à perdre : vise surtout la muscu, la balance bougera peu ».
Source : OMS (seuils IMC), HAS 2019 (repérage des TCA : question SCOFF), avis d'expert.

### D3. Aucune alerte quand le poids baisse trop vite
Constat : l'onglet Progrès alerte seulement si le poids **monte** de plus de 0,4 kg/semaine. Une perte de 1,5 kg/semaine (perte de muscle, signe possible de TCA ou de maladie) ne déclenche rien, même chez un ado.
Correction (l. 2792), avec `pct = trend / avg(w2) * 100` :
- `pct < -1` (toutes cibles) : « Tu perds plus de 1 % de ton poids par semaine : c'est trop rapide, tu risques de perdre du muscle. Ajoute environ 200 kcal par jour. »
- `pct < -1` et âge < 18, ou perte > 5 % en 1 mois : « Ton poids baisse vite. Parles-en à un adulte ou à ton médecin. »
- objectif muscle/force et `pct < -0.3` : « Ton poids baisse alors que tu veux prendre du muscle : ajoute 150 kcal par jour. »
Source : Helms 2014 (0,5-1 %/semaine), Garthe 2011.

### D4. Pas de seuil plancher ni de signal d'alerte chez les femmes (RED-S)
Constat : « ajuste de 150 kcal » sans limite. Une femme de 52 kg à 1 400 kcal qui retire 2 × 150 kcal arrive à 1 100 kcal, sous son métabolisme de base. Aucun mot sur les règles qui s'arrêtent.
Correction : calculer `floor = Math.max(bmr*1.1, f?1300:1600)` (avis d'expert) et l'afficher : « Ne descends pas sous **X kcal** par jour. » Ajouter pour les femmes dans le Guide, objectif gras : « Si tes règles deviennent irrégulières ou s'arrêtent, si tu es très fatiguée ou as souvent froid : arrête le régime, remonte tes calories et consulte. »
Ajouter une question (ou au moins un texte) : enceinte ou allaitante → pas de déficit, avis médical.
Source : IOC consensus RED-S 2023 (Mountjoy) ; énergie disponible < 30 kcal/kg de masse maigre.

## FAUX

### F1. Rythme de perte en kg fixes (0,3 à 0,7 kg/semaine)
Constat : pour une femme de 52 kg, 0,7 kg/sem = 1,35 %/sem (trop) ; pour un homme de 120 kg, 0,3 kg/sem = 0,25 % (trop lent, décourageant).
Correction (`bwNote`) : calculer à partir du poids : `lo=0.5%`, `hi=1%` du poids.
Texte : « Objectif : une baisse lente, entre **${lo} et ${hi} kg** par semaine. Regarde la moyenne, pas un jour isolé. » (ex. 75 kg → 0,4 à 0,75 kg). Pour IMC ≥ 30, garder 1 % max.
Source : ISSN 2017, Helms 2014.

### F2. Rythme de prise « 1 à 2 kg par mois » pour tous
Constat : 2 kg/mois pour une femme de 50 kg = 1 %/sem, c'est surtout du gras. Et +250 kcal/j ne donne de toute façon qu'environ +1 kg/mois : le texte et le calcul ne disent pas la même chose.
Correction : « Objectif : prendre doucement, environ **${0.25%}-${0.5%} kg par semaine** (débutant : haut de la fourchette ; femme ou confirmé : bas). » Ex. 75 kg → 0,2 à 0,4 kg/sem ; 55 kg → 0,15 à 0,3.
Source : Iraki 2019 (« Nutrition recommendations for bodybuilders in the off-season »), Helms 2014.

### F3. Alerte « Ça monte vite » identique pour tous
Constat : seuil fixe +0,4 kg/sem, quel que soit l'objectif, le poids ou l'âge. Elle conseille « enlève 150 kcal » à un ado en croissance et aux premières semaines de créatine. Pour « perdre du gras », toute hausse sur 2 semaines est déjà hors objectif.
Correction : seuil en % et selon l'objectif, sur la moyenne de 7 jours (déjà fait), avec au moins 2 semaines consécutives :
- gras : moyenne qui monte 2 semaines de suite → « Ta moyenne remonte : enlève 150 kcal par jour. »
- muscle/force : `pct > 0.5` → « Tu prends plus vite que prévu, c'est surtout du gras : enlève 150 kcal par jour. »
- forme : `pct > 0.5` → même texte.
- âge < 18 : jamais de conseil « enlève des calories » ; seulement l'alerte de baisse (D3).
- premières 3 semaines après le début de la créatine : ajouter « la créatine peut faire monter l'eau de 0,5 à 1 kg ».

### F4. Accueil Nils : « Au-dessus de ta zone » sur une seule pesée
Constat (l. 2158) : la comparaison à 70-71 kg utilise la dernière pesée (`lst`), pas la moyenne. Une pesée après un repas salé déclenche « enlève 150 kcal ». Le texte du Guide dit pourtant « regarde ta moyenne ».
Correction : utiliser la moyenne des 7 derniers jours (≥ 3 pesées) au lieu de `lst`.

### F5. Surplus calorique donné aux personnes en surpoids (muscle, force)
Constat : homme 35 ans, 190 cm, 120 kg (IMC 33), objectif force → 3 550 kcal ; homme 100 kg IMC 29, muscle → 3 450 kcal. Chez un débutant en surpoids, la recomposition (maintien ou léger déficit) fait prendre du muscle sans ajouter de gras.
Correction :
```js
if((p.goal==="muscle"||p.goal==="force") && bmi>=27) adj = bmi>=30 ? -0.10*tdee : 0;
```
Texte : « Tu peux prendre du muscle tout en perdant un peu de gras : on mange autour du maintien. »
Source : Barakat 2020 (recomposition), avis d'expert. Note : IMC n'est qu'un repère ; proposer « Je suis très musclé(e) » (case) pour annuler cette règle.

## AMÉLIORABLE

- **A1. Plafond 160 g de protéines.** Arbitraire : un homme sec de 100 kg / 1,90 m (IMC 27,7) est coupé à 160 g (1,6 g/kg), ce qui reste correct ; mais combiné au calcul IMC 25, un homme musclé de 100 kg / 1,85 m n'a que 135-160 g. Remplacer par `hi = Math.round(pw*2.0/5)*5` sans plafond fixe, et seuil de bascule IMC > 30 (au lieu de 27) pour les hommes. Source : ISSN 2017.
- **A2. Protéines plus hautes en déficit** quand on est mince : objectif gras et IMC < 25 → 1,8 à 2,2 g/kg (protège le muscle). Source : Helms 2014 (2,3-3,1 g/kg de masse maigre).
- **A3. Seniors (≥ 60 ans), objectif forme** : 1,4-1,8 g/kg est bien ; ajouter « au moins 25-30 g de protéines à chaque repas ». Source : PROT-AGE 2013, ESPEN 2014.
- **A4. Repas post-séance** : écrire « 0,3 à 0,4 g de protéines par kg » ou adapter : `Math.round(w*0.3)`-`Math.round(w*0.4)` g. Ajouter une option sans lactose : « une boîte de thon avec du riz ». Source : ISSN 2017. « Dans les 2 heures » n'est pas indispensable si un repas protéiné a eu lieu 2-3 h avant : écrire « au repas suivant ou dans les 2 heures ».
- **A5. Créatine** : n'existe que dans le Guide de Nils. Pour les autres adultes, l'ajouter en option : « Facultatif : créatine monohydrate 3 à 5 g par jour. Bois normalement. Si tu as un problème aux reins, demande d'abord à ton médecin. » **Jamais** affichée avant 18 ans (ANSES 2016 et Code du sport : déconseillée aux mineurs ; ISSN la juge sûre mais la vente/conseil aux mineurs n'est pas recommandé en France).
- **A6. « Ne descends jamais très bas »** (Guide, gras) : trop vague. Remplacer par le plancher chiffré de D4.
- **A7. Fréquence de pesée** : « 3 fois par semaine » est bien pour un adulte. Pour < 18 ans ou IMC < 18,5 : proposer de masquer la courbe et de ne pas suggérer la pesée (« suis plutôt tes progrès en séance et tes photos »). Ajouter un réglage « Ne pas afficher mon poids ». Avis d'expert (prévention TCA).
- **A8. Mots** : « brûler un maximum » (intro de séance gras) et « tu brûles plus » : remplacer par « garder ton muscle et rester en mouvement ». Évite d'associer la séance à une punition calorique. Avis d'expert.
- **A9. Lipides et glucides** : absents. Pas indispensable. Si ajoutés : lipides ≥ 0,8 g/kg (et ≥ 20 % des calories, ≥ 25 % chez la femme), glucides = le reste, avec au moins 3 g/kg les jours de séance. Source : ISSN 2017, ANSES 2016 (lipides 35-40 %).
- **A10. Âge minimum 12 ans** : passer à 14 ans minimum, ou afficher « moins de 14 ans : demande l'accord d'un parent » (en plus de D1).
- **A11. Arrondi « 1 550 kcal »** affiché « ≈ » sur le Guide mais sans « ≈ » sur l'écran final : afficher partout « environ ».

## Deux objectifs combinés : comment calculer

Règle générale : les calories suivent l'objectif qui demande **le plus de prudence** ; les protéines prennent **la fourchette la plus haute** des deux. Toujours appliquer d'abord D1 (mineurs), D2 (maigreur), F5 (surpoids), D4 (plancher).

Notation : `T` = maintien (Mifflin × activité), `pw` = poids de calcul (poids réel, ou poids à IMC 25 si IMC > 30 ; 27 pour les femmes).

| Combinaison | Calories | Protéines | Rythme affiché |
|---|---|---|---|
| Gras + Muscle (recomposition) | IMC ≥ 25 : `T − 10 %` · IMC 20-25 : `T − 5 %` (≈ maintien) · IMC < 20 : `T` (pas de déficit) | 2,0 à 2,4 g/kg `pw` | 0 à −0,5 %/sem ; « le poids bouge peu, regarde le tour de taille et les charges » |
| Gras + Force | comme gras seul, mais déficit limité à `T − 15 %` | 1,8 à 2,2 g/kg | −0,5 à −0,8 %/sem |
| Gras + Forme | `T − 15 %` (IMC < 20 : `T`) | 1,6 à 2,0 g/kg | −0,5 à −0,8 %/sem |
| Muscle + Force | `T + 150 (F)` / `T + 250 (H)` ; IMC ≥ 27 : `T` | 1,6 à 2,0 g/kg | +0,25 à +0,5 %/sem |
| Muscle + Forme | `T + 100 (F)` / `T + 200 (H)` ; IMC ≥ 27 : `T` | 1,6 à 2,0 g/kg | +0,25 %/sem max |
| Force + Forme | `T` (maintien) | 1,4 à 1,8 g/kg | stable |

Pour les mineurs, toutes les combinaisons : pas de chiffre de calories (« mange à ta faim »), protéines 1,2 à 1,6 g/kg, et « perdre du gras » ne donne aucun déficit.

Code proposé :
```js
function adjFor(goals,T,f,bmi,age){
  if(age<18) return 0;
  const g=new Set(goals);
  if(g.has("gras")){
    if(bmi<18.5) return 0;                       // bloqué en amont
    if(g.has("muscle")) return bmi>=25?-0.10*T:bmi>=20?-0.05*T:0;
    const d=g.has("force")?0.15:g.has("forme")?0.15:0.17;
    return -(bmi<20?0.10:d)*T;
  }
  if(g.has("muscle")||g.has("force")){
    if(bmi>=27) return bmi>=30?-0.10*T:0;
    if(g.has("muscle")&&g.has("forme")) return f?100:200;
    return f?150:250;   // muscle, force, ou muscle+force (force seule : 150)
  }
  return 0;             // forme, force+forme
}
const PROT={gras_muscle:[2.0,2.4],gras_force:[1.8,2.2],gras_forme:[1.6,2.0],
  muscle_force:[1.6,2.0],muscle_forme:[1.6,2.0],force_forme:[1.4,1.8]};
```
Sources : Barakat 2020 (recomposition), Helms 2014, ISSN 2017 (protéines ; régimes et composition corporelle), avis d'expert pour les pourcentages exacts.

## Résultats chiffrés (extrait)

| Profil | kcal | kcal/BMR | Protéines | Problème |
|---|---|---|---|---|
| F 22 ans 165 cm 45 kg, gras | 1 550 | 1,28 | 70-90 g | IMC 16,5 : déficit (D2) |
| F 14 ans 155 cm 45 kg, gras | 1 600 | 1,35 | 70-90 g | mineure en déficit (D1) |
| H 12 ans 150 cm 40 kg, muscle | 2 150 | 1,68 | 65-80 g | 12 ans accepté, Mifflin non valide (D1, A10) |
| H 16 ans 170 cm 85 kg, gras | 2 250 | 1,22 | 115-145 g | mineur en déficit (D1) |
| F 30 ans 160 cm 52 kg, gras | 1 400 | 1,16 | 85-105 g | pas de plancher affiché (D4) |
| H 35 ans 190 cm 120 kg, force | 3 550 | 1,60 | 145-160 g | surplus en obésité (F5) |
| H 30 ans 185 cm 100 kg, muscle | 3 450 | 1,72 | 135-160 g | surplus à IMC 29 (F5), plafond (A1) |
| F 45 ans 160 cm 110 kg, gras | 2 000 | 1,17 | 100-130 g | correct |
| H 25 ans 180 cm 75 kg, gras | 2 150 | 1,23 | 120-150 g | correct |
| F 60 ans 158 cm 70 kg, forme | 1 750 | 1,43 | 85-110 g | correct |

Tableau complet (28 profils) : `res.tsv`.

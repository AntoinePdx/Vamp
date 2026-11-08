# Vamp — notes pour Claude

Web app iPhone (PWA) pour composer avec des suites d'accords. Hébergée sur GitHub Pages, installée via Safari → « Sur l'écran d'accueil ».

## Ce que fait l'app
- Choix d'un style (R&B, Neo-soul, Disco, House), d'une tonalité (12 notes) et d'un mode (Maj / Min).
- Une suite de 4 accords (pads bleus numérotés 1 à 4) + 12 autres accords du style en dessous.
- On **tient** un pad pour jouer l'accord, on relâche pour l'arrêter. Pas de séquenceur ni de lecture auto : c'est voulu.
- « Generate » tire une nouvelle suite du même style et du même mode.
- En haut : l'accord joué en grand + ses notes. À droite, 4 tuiles son : Sound (menu Juno / Rhodes / Orgue / Cordes / Pluck), FX (chorus), Filter, Release.
- Bloc style + tonalité repliable via une ligne de résumé (Hide / Edit), état mémorisé.
- « Share MIDI » : fichier .mid (accords plaqués, 1 mesure par accord, 110 BPM fixe, piste accords + piste basse, mêmes voicings que ceux qu'on entend) → menu de partage iOS (`navigator.share` avec un `File`). Sinon, téléchargement direct du .mid.
- Interface en anglais. L'option de notation « Do Ré Mi » reste en français.

## Fichiers
- `index.html` : toute l'app (HTML + CSS + JS vanilla dans une IIFE).
- `sw.js` : service worker (hors connexion, cache puis mise à jour en arrière-plan).
- `manifest.webmanifest`, `icons/` : installation et icônes. Logo : 5 notes sur une portée qui dessinent un V.
- `.nojekyll` : pour GitHub Pages.

## Organisation du JS (index.html)
- **Théorie** : `Q` (qualités d'accords → intervalles + symboles), `STYLES` (suites en degrés par style et par mode), `info(tok)` qui transforme un jeton en accord concret.
  - Notation des suites : `'ii:m9 V:13 I:maj9 vi:m9'`, `degré:qualité`, `@degré` = accord sur basse (ex. `IV:maj9@V`).
  - Les 12 « autres accords » = les jetons les plus fréquents des suites du style, sans ceux de la suite en cours, triés par degré (`otherChords`).
- **Conduite des voix** : voicings sans fondamentale pour les accords de 4 sons et plus (la basse joue la fondamentale). On teste tous les renversements et on garde celui qui bouge le moins, registre Mi3 à Do6 (MIDI 52 à 84) (`candidates`, `moveCost`, `voiceSuite`).
- **Rendu** : `renderSelectors`, `renderPads`, `showChord`, `renderSound`.
- **Synthé Web Audio** : `PRESETS` (soustractif pour Juno, Cordes, Orgue ; FM pour Rhodes et Pluck), `ensureAudio` (chaîne compresseur, chorus stéréo, réverbe à convolution), `voice` / `release`.
  - Astuce mode silencieux iPhone : `navigator.audioSession.type = 'playback'` + boucle audio muette (`silentWav`). Ne pas retirer.
- **Jeu** : événements pointer sur les pads (`padOn` / `padOff`, pointer capture).
- **MIDI** : `buildMidi` (format 1, 3 pistes, PPQ 480), `shareMidi`.
- **Préférences** : localStorage, clés `atelier-accords-v3` (style, mode, tonalité, notation, son, curseurs) et `atelier-accords-picker`. Ne pas renommer ces clés sans migration, sinon les réglages des utilisateurs sont perdus.

## Choix techniques à respecter
- Un seul `index.html`, JS vanilla. Pas de framework, pas de build, pas de dépendances (sauf Google Fonts : Geist, Geist Mono).
- Son 100 % Web Audio, pas d'échantillons.
- Pas de compte ni de base de données ; on a volontairement abandonné l'enregistrement des suites. Seules les préférences vont en localStorage.
- **À CHAQUE modification d'un fichier de l'app, incrémenter `VERSION` en haut de `sw.js`** (`vamp-v7` → `vamp-v8`…), sinon les iPhone gardent l'ancienne version en cache.
- Si on ajoute un fichier à l'app, l'ajouter aussi à `APP_FILES` dans `sw.js`.

## Design
- Thème clair unique. Bleu cobalt `#2f4fd8`, encre `#1d1e22`, fond `#fafafb`, filets gris clair, interface aérée.
- Le CSS est empilé en couches : les règles de la fin (« Finitions », « Version aérée ») écrasent celles du début. Vérifier la dernière règle qui s'applique avant de modifier un style.
- **iPhone SE (375×667)** : tout doit tenir sans défiler, du haut jusqu'aux 4 pads de la suite.
- Aucun texte ne doit déborder ni se chevaucher, même avec les noms les plus longs en notation Do Ré Mi (ex. « Si♭maj7♯11/Do »).

## Façon de travailler avec l'utilisateur
- Expliquer en français, simplement : l'utilisateur est musicien (théorie niveau intermédiaire), pas développeur pro.
- Avant une modification importante, résumer en 2 ou 3 lignes ce qui va être fait.
- Changer seulement ce qui est demandé ; ne pas casser ce qui marche.
- Après chaque changement : vérifier que la page se charge sans erreur console et que la mise en page tient en 375×667 (navigateur headless installé dans le scratchpad, jamais dans le projet). Dire ensuite ce qu'il faut tester sur l'iPhone : le son et le menu de partage ne se testent bien que sur l'appareil.
- Un commit Git à chaque étape qui fonctionne, message clair en français.

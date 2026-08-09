# Mémoire de projet — Yaniv (suivi de score)

*Dernière mise à jour : 9 août 2026 — version courante : **`index.html` (v10, publié = Yaniv Club)** ; archives dev = `dev/Yaniv v9.html` (merge Damien), `dev/Yaniv v10.html` (sauvegarde+QR).*

## ⭐ v10 (09/08) — sauvegarde renforcée hors-ligne + transfert par QR (choix « sans compte »)
Thomas a signalé que le site publié « n'enregistre pas les parties » (localStorage = par navigateur/appareil, effacé en privé / purge iOS). Choix retenu : **rester 100 % autonome, sans backend/compte**, et renforcer :
- **Anti-perte local** : miroir de secours `yaniv_v5_bak` écrit à chaque `persist()` ; `loadLibrary` retombe dessus via `parseLib()` si la clé principale est corrompue. Rappel « dernière sauvegarde fichier » (`yaniv_last_export`, `updateBackupInfo()`), bouton **💾 Sauvegarder sur l'appareil** (export `.json` = la vraie copie qui survit à une purge). Note iOS (ajouter à l'écran d'accueil + exporter).
- **Transfert par QR** : lib **qrcode-generator (Arase, MIT) minifiée ~21 Ko inlinée** dans un `<script>` dédié avant le principal. `qrSVG()` rend un QR SVG noir/blanc (marge 4), **cappé à ≤105 modules** (scannable) sinon repli lien. Modale `#shareModal` : QR + « Envoyer » (navigator.share) + « Copier le lien ».
- **⚠️ Payload COMPACT v2 obligatoire pour le QR** : le format brut est inQRable (3 manches = 137 modules ; 8+ manches dépassent la capacité). `encodeShare()`/`decodeShare()` : `{v:2,n,p[],s[],r,h[[score,flags]],f}` — manches en tableaux, drapeaux asaf/yaniv en bitmask ; bonus/stats/events reconstruits par `recompute()` à l'import. Résultat : 15 manches ≈ 101 modules → OK. Rétro-compatible avec l'ancien `{name,state}`. `importPayload`/`checkSharedLink` passent par `decodeShare` puis `recompute`.
- Vérifié : round-trip encode→décode→recompute = mêmes scores ; QR petite partie OK / énorme payload → null (repli) ; import compact ajoute bien la partie. (harness DOM + node --check.)



> **Organisation des dossiers (09/08) — un seul dossier `yaniv-club`.** Le produit publié est `index.html` (GitHub Pages `coretc.github.io/yaniv-club/` + Gitea). Tout l'historique de dev (v4→v8, le v7.1 de Damien, cette mémoire, l'archive v9) vit dans `dev/`, versionné sur Gitea **et** GitHub mais sans affecter le site (Pages ne sert que `index.html` à la racine). L'ancien dépôt Gitea `Yaniv` reste comme sauvegarde dormante.

## ⭐ v9 (09/08) — reprise des améliorations de Damien (`Yanov_v7_1.html`) dans le Club
Damien avait branché sur `Yaniv v7.html` ; ses apports ont été **portés et adaptés** aux abstractions v8 (`RULES().limit`, `isPalier()`, mode édition `editIndex`) — pas collés bruts. Vérifié par simulation Node (22 assertions) + parcours DOM complet du vrai script.
- **☠️ Élimination progressive (le gros changement de règle)** : dépasser la limite **élimine** le joueur mais la partie **continue** ; elle ne finit que lorsqu'il ne reste qu'**un survivant**. Helpers `survivors()` + `isGameOver()` (= un joueur a dépassé ET survivants ≤ 1), centralisés dans `checkEnd`/`commitRound`/`saveRules`/`deleteEditedRound`/`undoRound` (fini le `some(>limit)` éparpillé).
- **Survivants uniquement** : `draftPlayers` (nouvelle var) = survivants pour une nouvelle manche (tous en édition) ; `renderDraft`/`commitRound` itèrent `draftPlayers`. Les éliminés ne saisissent plus et sont exclus de la distribution (`dealerPool()` filtre par limite, `pickDealer`/`advanceDealer`/`ensureDealer` adaptés).
- **🃏 Distributeur = ROTATION** (décision confirmée 09/08, on NE prend PAS le random-chaque-manche de Damien) : aléatoire au départ, puis `advanceDealer()` tourne dans l'ordre parmi les survivants.
- **🎖️⚔️⤵️ `roundDetailsHTML()`** : bloc « Détail par manche » (Yaniv/Asaf/Palier) ajouté au bilan.
- **Graphe** : graduation des manches en abscisse + label « MANCHES », marge basse 36, lignes de palier/défaite bornées à `RULES().limit` (au lieu de 200 en dur).
- **UX** : `loadLibrary`/`deleteGame` ne créent plus de partie vide d'office (lobby propre) ; `removePlayer` autorise le retrait du **dernier** joueur (vide la partie).
- `nextPalier` allait déjà jusqu'à la limite en v8 → rien à porter.



## ⭐ v7 « Club » — refonte complète (ne reprend pas la base v4/v5/v6)

Fichier mono-page repensé en **app 3 écrans** : Lobby (bibliothèque) → Table (partie live, onglets Classement/Historique/Stats) → **feuille de manche avec pavé numérique tactile** (bottom-sheet). Thème « club de cartes » (noir/or/émeraude, wordmark serif). Même couche de données que v5/v6 (clé `yaniv_v5`, migration v4 conservée).

### ⚠️ Correctif critique iOS (08/08)
`prompt()`/`confirm()`/`alert()` sont **bloqués silencieusement quand l'app tourne en mode écran d'accueil (standalone) sur iPhone** (Edge/Safari WebKit) → supprimer joueur / nouvelle partie / renommer ne faisaient RIEN. Remplacés par modales maison `uiAlert`/`uiConfirm`/`uiPrompt` (Promise). v5/v6 ont encore le bug → **n'utiliser que v7**. Voir mémoire [[feedback-artifact-sandbox-dialogs]].

### Fonctionnalités ajoutées 08/08 (toutes dans v7)
- **Distributeur / mélangeur (tour de roll)** : `state.dealer`, tirage de départ **aléatoire** (`pickDealer`), rotation à chaque manche (`advanceDealer`), bouton 🎲 pour re-tirer. Affiché en HUD + badge « 🃏 donne » sur la ligne du joueur.
- **Prochain palier −50** : `nextPalier(score)` → affiché sous chaque joueur du classement (« 🎯 −50 dans X pts »).
- **Annonce vocale de fin de manche** : Web Speech API (`speechSynthesis`, fr-FR), toggle 🔊/🔇 en barre du haut (pref `yaniv_voice`). Annonce scores à chaque manche + gagnant/perdant à la fin.
- **Fin de partie à >200 (≥201)** : `LIMITE=200`, `checkEnd()` — celui qui dépasse 200 est **éliminé (perd)**, le plus bas gagne. `state.finished`/`endedAt`. Barre d'action bascule en « 🏆 Bilan / 🔁 Revanche ». `undoRound` ré-ouvre la partie si on repasse sous 200.
- **Bilan de fin de partie** : modale (gagnant, éliminé(s), durée, nb manches, classement final).
- **CSV supprimé → remplacé par un graphe « Rapport victoire / défaite »** : SVG maison sans dépendance (trajectoire cumulée des scores par manche, ligne rouge pointillée à 200, légende). `buildSeries()`/`chartSVG()`/`renderReport()`.
- **Timer de partie non handicapant** : `state.startedAt` (1re manche) → `endedAt`, affiché en HUD (`MM:SS`), s'arrête quand un joueur dépasse 200.
- **Revanche** (`rematch`) : nouvelle partie avec les mêmes joueurs.

Logique de score vérifiée par simulation Node 08/08 (palier −50, pas de cascade sur 0, fin >200, série graphe alignée).

### Ajouts 08/08 (lot « faire mieux » — Damien/Fersi)
- **Distributeur** : aléatoire UNIQUEMENT au départ (`pickDealer`/bouton 🎲), puis **rotation dans l'ordre d'ajout** (`advanceDealer`) à chaque fin de manche (Damien voulait une logique, pas de l'aléatoire permanent). Annonce vocale « qui distribue » **après validation des points** seulement, et pas si quelqu'un a perdu.
- **Voix féminine** : `loadVoices()` choisit une voix FR féminine (regex noms), sinon féminine autre langue ; **sélecteur de voix** dans l'onglet Stats (`voiceSel`, pref `yaniv_voice_uri`) + bouton Tester ; débit 0.85/pitch 1.15. ⚠️ sur iPhone sans voix FR féminine installée → « Thomas » (homme) ; l'utilisateur doit installer Aurélie (Réglages ▸ Accessibilité ▸ Contenu énoncé ▸ Voix).
- **Règles configurables par partie** (`state.rules` = {limit, asaf, paliers, yanivZero}) via `RULES()`/`defaultRules()` + modale ⚙️. La constante `LIMITE` a été remplacée partout par `RULES().limit`.
- **⚠️ Palier −50 désormais DÉRIVÉ dans `recompute()`** (plus lu depuis le flag stocké) — sinon éditer une manche antérieure laissait les paliers suivants figés (bug A=−35 attrapé en test). Rend aussi le toggle « paliers » rétroactif.
- **Éditer/supprimer n'importe quelle manche** : lignes de l'Historique cliquables → feuille pré-remplie (`editIndex`, `scoresBefore(idx)` pour projeter), bouton 🗑️ ; ré-évalue la fin de partie dans les deux sens.
- **Palmarès du groupe** (`computePalmares`) : victoires/défaites/parties/moyenne par nom sur TOUTES les parties (bouton dans le Lobby).
- **Sauvegarde & partage sans serveur** : Export/Import `.json` de toute la bibliothèque + **partage par LIEN** (08/08, remplace l'ancien code copier/coller jugé confus) : `shareGame()` construit `…/#g=<base64>` et utilise `navigator.share` (menu natif iOS/Android) sinon copie le lien ; `checkSharedLink()` au démarrage propose d'importer une partie reçue par lien ; `importCode()` accepte lien, `#g=` ou ancien `YANIV1:`. ⚠️ QR volontairement NON fait (encodeur fiable trop lourd + hors-ligne).
- **Finitions** : haptique `navigator.vibrate` (validation/palier/défaite), **confettis** canvas maison à la victoire, **manifest + apple-touch-icon** injectés au runtime (`installMeta`, icône ♠ or) pour « ajouter à l'écran d'accueil ».
- Toutes les nouvelles logiques vérifiées par simulation Node 08/08 (édition, suppression, paliers off, palmarès, round-trip base64).
- **Règle palier affinée (08/08)** : paliers = TOUS les multiples de 50 jusqu'à la limite INCLUSE (50/100/150/**200**), via `isPalier(cum)= paliers && cum>0 && cum%50===0 && cum<=limit`. Atterrir **pile sur 200 → −50 → 150** ; on ne perd QUE si on **dépasse** (201+, `sc>limit`). `nextPalier` va jusqu'à la limite ; message « plus de palier » supprimé. Vérifié : 190+10=150, 190+15=205→perdu.

---

## Historique v4→v6 (superseded par v7)
*Version courante précédente : `Yaniv v5.html`*

## Vue d'ensemble

Application web mono-fichier pour suivre le score d'une partie de Yaniv. Pas de dépendances externes, pas de backend : tout vit dans un seul fichier HTML/CSS/JS, avec sauvegarde automatique dans `localStorage`.

- **v4** (`Yaniv v4.html`) : mono-partie, orientée iPad/Safari (clé `yaniv_pro_data`). Conservée.
- **v5** (`Yaniv v5.html`) : reprise du projet (« Yaniv reprend ici »). Deux axes ajoutés :
  1. **Compatibilité tout mobile / tout navigateur** : police avec fallbacks (Segoe UI, Roboto, Noto), viewport qui **autorise le zoom** (`user-scalable=no` retiré → accessibilité) mais inputs en 16px pour éviter l'auto-zoom iOS ; `-webkit-appearance` sur boutons, `-webkit-text-size-adjust`, `theme-color`, media query ≤480px, historique scrollable en X, flex-wrap partout, thème sombre persistant (`yaniv_dark`).
  2. **Bibliothèque multi-parties** (le vrai ajout) : enregistrer / démarrer une nouvelle / reprendre / renommer / supprimer plusieurs parties.

## Modèle de données v5 (bibliothèque)

- Clé `localStorage` = `yaniv_v5` → `{ currentId, games:{ [id]: {id, name, createdAt, updatedAt, state} } }`.
- `state` = l'ancien objet `{players, scores, history, events, stats}` inchangé (logique de jeu identique à v4).
- **Migration auto** : au 1er lancement, si `yaniv_pro_data` (v4) existe, il est importé comme « Partie récupérée ».
- L'état de travail `state` est **la même référence** que `games[currentId].state` ; chaque `save()` (aliasé sur `persist()`) réécrit le slot + `updatedAt` puis persiste toute la bibliothèque.
- Fonctions ajoutées : `loadLibrary`, `activate`, `createGame`, `newGame`, `loadGame` (« Reprendre »), `deleteGame`, `renameGame`/`renameCurrent`, `openGamesModal`/`closeGamesModal`/`renderGamesModal`, `updateTitle`, `uid`, `blankState`, `sanitizeState`, `defaultName`, `fmtDate`.
- UI : barre du haut avec **titre cliquable** (= renommer) + bouton 📂 Parties ; modale HTML maison (overlay) listant les parties (joueurs / manches / date maj, boutons ▶️ Reprendre ✏️ 🗑️). « ➕ Nouvelle partie » ne détruit plus rien : l'ancienne reste enregistrée.
- Dialogues : `prompt`/`confirm` natifs (OK en navigateur réel autonome — la restriction connue ne vaut que pour le sandbox d'artifact Claude).

## Architecture

- **État global** : un objet `state` unique `{ players, scores, history, events, stats }`
  - `players` : tableau de noms (strings)
  - `scores` : `{ nom: score_cumulé }`
  - `history` : tableau de manches, chaque manche est `{ nom: {score, asaf, yaniv, bonus} }`
  - `events` : journal chronologique (max 50 entrées) affiché dans "📋 Événements"
  - `stats` : `{ nom: {yaniv, asaf, bonus} }` — compteurs cumulés
- **Fonctions clés** : `addPlayer`, `removePlayer`, `saveRound`, `undoRound`, `newGame`, `recompute` (recalcule tout depuis `history`), `render` (redessine le DOM), `exportCSV`
- **Rendu** : délégation d'événements sur `#players` et `#roundInputs` (boutons dynamiques identifiés par `data-index`, pas par nom interpolé — voir correctifs de sécurité ci-dessous)

## Règles de jeu codées

- Un joueur "Asaf" reçoit `+30` sur les points de sa main
- Un palier bonus de `-50` s'applique quand le score cumulé **atteint exactement** 50, 100 ou 150 **via l'apport de la manche en cours** (voir correctif du 7 août ci-dessous)
- Pas de logique d'élimination automatique à 100/200 points — l'appli ne "sort" jamais un joueur automatiquement

## Historique des correctifs

1. **Erreurs de syntaxe** — accents graves manquants sur les template literals (`confirm(...)`, `text:...`) → corrigés.
2. **`removePlayer` défini deux fois, dont une fois imbriqué dans `render()`** → le bouton "Supprimer" ne fonctionnait pas du tout (portée de fonction locale, inaccessible depuis `onclick` global). Déplacé en fonction unique au niveau racine.
3. **Confirmation du comportement "dernier joueur"** — l'alerte "Impossible de supprimer le dernier joueur" est un garde-fou voulu, pas un bug.
4. **"Nouvelle partie" ne vidait pas la liste des joueurs** → corrigé pour repartir sur une liste de joueurs entièrement vide.
5. **Faille d'injection HTML/attribut** — un nom contenant une apostrophe (ex. `O'Brien`) cassait le bouton Supprimer ; un nom contenant `<b>` pouvait injecter du HTML dans la page. Corrigé en :
   - remplaçant les `onclick` avec nom interpolé par une délégation d'événements basée sur `data-index`
   - ajoutant `escapeHtml()`, utilisé partout où un nom/texte est inséré dans le DOM
6. **Récupération après `localStorage` corrompu** — `JSON.parse` sans `try/catch` plantait l'appli au chargement. Corrigé avec repli propre sur un état vide + message d'alerte.
7. **Noms réservés bloqués** (`__proto__`, `constructor`, `prototype`) pour éviter toute corruption des objets internes.
8. **Doublons insensibles à la casse** — "Alice" et "alice" sont maintenant détectés comme le même joueur.
9. **Journal d'événements plafonné à 50 entrées** pour éviter une croissance illimitée du `localStorage`.
10. **Égalités au classement** — auparavant seul le premier joueur (ordre alphabétique) d'un score à égalité était surligné en tête/dernier ; corrigé pour surligner tous les joueurs à égalité.
11. **Échappement CSV** — les noms contenant `;`, `"` ou retour à la ligne corrompaient l'export ; corrigé selon la norme CSV.
12. **Confort de saisie** — touche Entrée pour ajouter un joueur, focus auto-remis sur le champ.
13. **Garde-fou `saveRound()`** si aucun joueur n'est présent.
14. **Bug du bonus en cascade (7 août)** — le bonus `-50` se déclenchait à nouveau quand un joueur *restait* exactement sur un seuil (ex. 50, après un bonus précédent depuis 100) et entrait 0 point à la manche suivante, car `future = score_actuel + 0 = score_actuel` retombait pile sur un seuil. Cela pouvait faire fondre le score en cascade (150→100→50→0) sur des manches à 0 point consécutives. **Corrigé** : le bonus n'est désormais appliqué que si la manche apporte réellement des points (`score > 0`) en plus d'atteindre exactement 50/100/150.

## Hypothèses / points ouverts (non tranchés avec l'utilisateur)

- **200 points** : je pars de l'hypothèse classique où 200 est un seuil d'élimination (fin de partie pour ce joueur), pas un palier bonus supplémentaire — mais **cette logique d'élimination n'est pas implémentée du tout** dans le code actuel. À confirmer si souhaitée.
- **Bouton "0" (quickZero)** : coche actuellement la case Yaniv en même temps qu'il met les points à 0. Dans les règles officielles, annoncer Yaniv n'implique pas forcément un score de 0 (juste la main la plus basse). Ce couplage n'a pas été changé, faute de demande explicite — à signaler si ça pose problème en pratique.

## Pistes d'amélioration possibles (non demandées, juste notées)

- Séparer le bouton "0 points" de la case "Yaniv"
- Ajouter une logique d'élimination configurable (100 ou 200 points)
- Permettre de renommer un joueur plutôt que devoir le supprimer/ré-ajouter

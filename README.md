# 🏆 Yaniv Club

Application web **mono-fichier** pour suivre le score d'une partie de **Yaniv** — sans compte, sans serveur, hors-ligne, sur n'importe quel téléphone ou navigateur.

👉 **Jouer / ouvrir** : télécharger `index.html` et l'ouvrir dans un navigateur (le lien direct GitHub Pages sera ajouté ici une fois activé).

## Fonctionnalités

- 📚 **Bibliothèque de parties** : enregistrer, reprendre, renommer, supprimer plusieurs parties (sauvegarde auto dans le navigateur).
- 🃏 **Distributeur** : tiré au sort au départ, puis rotation dans l'ordre des joueurs.
- 🎯 **Paliers −50** automatiques (50 / 100 / 150 / 200). Atterrir pile sur 200 redescend à 150 ; on ne perd qu'en **dépassant** 200.
- 🗣️ **Annonce vocale** des scores et du distributeur (voix féminine, sélecteur de voix).
- ✏️ **Correction** de n'importe quelle manche, ↩️ annulation, ⏱️ chrono de partie.
- 📈 **Rapport** graphique (trajectoire des scores) + 🏁 bilan de fin de partie.
- 🏆 **Palmarès du groupe** (victoires / défaites / moyenne, toutes parties confondues).
- ⚙️ **Règles configurables** (limite, valeur de l'Asaf, paliers, « Yaniv = 0 »).
- 💾 **Sauvegarde & partage** : export/import `.json`, code de partage entre téléphones.
- 🎉 Confettis, retour haptique, installable sur l'écran d'accueil.

## Technique

Un seul fichier `index.html` (HTML + CSS + JS, zéro dépendance). Les données restent **en local** dans le `localStorage` de ton navigateur — rien n'est envoyé nulle part.

## Licence

MIT — fais-en ce que tu veux.

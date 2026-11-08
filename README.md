# Vamp

![Vamp](icons/logo.svg)

Play R&B, neo-soul, disco and house chords with your fingers, and share the progression as a MIDI file.
A vamp is a short chord progression repeated in a loop.

## Contenu

| Fichier | Rôle |
| --- | --- |
| `index.html` | L'appli complète |
| `sw.js` | Mode hors connexion |
| `manifest.webmanifest`, `icons/` | Icône et installation sur l'écran d'accueil |

Pas de compte ni de base de données : tout se passe sur le téléphone.

## Mettre en ligne

1. Crée un dépôt GitHub public (par exemple `vamp`) et envoie ces fichiers à la racine.
2. Settings → Pages → Deploy from a branch → `main` / `(root)` → Save.
3. Sur l'iPhone, ouvre l'adresse GitHub Pages dans Safari → Partager → Sur l'écran d'accueil.

## Mettre à jour

Remplace les fichiers, puis change `VERSION` en haut de `sw.js` (par exemple `vamp-v7`).

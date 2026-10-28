# Kart Chrono

Mini-jeu de karting contre-la-montre (three.js), jouable au téléphone, installable comme application
(PWA) et jouable **hors ligne**.

**Jouer :** https://william-prove.github.io/kart-chrono/

## Sur iPhone / iPad

1. Ouvrir le lien dans **Safari**, téléphone à l'horizontale.
2. Bouton Partager → **« Sur l'écran d'accueil »** : le jeu s'ouvre ensuite en plein écran.
3. Au premier départ, accepter **« Mouvement et orientation »** pour piloter à l'inclinaison
   (impossible sans HTTPS ; les boutons tactiles restent disponibles).

## Sur Android

Ouvrir le lien dans Chrome → menu → **« Installer l'application »** (ou « Ajouter à l'écran d'accueil »).

## Contenu

```
site/                 ce qui est publié (site statique, aucun backend)
  index.html          le jeu (HTML + CSS + JS dans un seul fichier)
  sw.js               service worker : cache hors ligne
  manifest.webmanifest
  vendor/             three.js r128 + GLTFLoader, servis en local
  models/pilote.glb   modèle 3D du pilote
  icons/              icônes 180 / 192 / 512
  fonts/              polices Saira téléchargées en local (aucun appel à Google)
.github/workflows/    publication automatique sur GitHub Pages
```

Les chronos et le fantôme sont enregistrés dans le navigateur du téléphone (`localStorage`) :
ils survivent aux mises à jour, tant que l'URL ne change pas.

## Mettre à jour le jeu

Remplacer le contenu de `site/` (tout le dossier, y compris `vendor/`, `models/`, `fonts/`), puis :

```bash
git add -A && git commit -m "maj du jeu" && git push
```

Le workflow republie automatiquement. `index.html` est revalidé à chaque ouverture (le service
worker interroge le réseau en premier) : le téléphone récupère la nouvelle version au prochain
chargement.

## Tester en local

```bash
cd site && python3 -m http.server 8123
# puis http://127.0.0.1:8123/
```

Un serveur local est nécessaire (les modules 3D et le service worker ne fonctionnent pas en `file://`).

# Kart Chrono — déploiement sur le VPS

Instructions pour Hermès. Le jeu est un **site statique** : aucun backend, aucune base de données, aucune variable d'environnement. Les chronos et le fantôme sont enregistrés dans le navigateur du téléphone (localStorage).

## Contenu du zip

```
DEPLOY.md                  ce fichier
site/
  index.html               le jeu (HTML + CSS + JS dans un seul fichier)
  vendor/three.min.js      moteur 3D (three.js r128), servi en local
  vendor/GLTFLoader.js     chargeur des modèles 3D
  models/pilote.glb        modèle 3D du pilote (~0,6 Mo) ; le kart est dessiné dans index.html
  manifest.webmanifest     pour « Ajouter à l'écran d'accueil »
  icons/                   icônes 180, 192, 512 px
```

Seul le dossier `site/` doit être servi. Tous les chemins sont relatifs : le jeu fonctionne à la racine d'un domaine (`https://kart.exemple.be/`) comme dans un sous-dossier (`https://exemple.be/kart/`).

## Exigence obligatoire : HTTPS

Sur iPhone, Safari ne donne accès au capteur d'inclinaison (`DeviceMotionEvent.requestPermission`) que sur une page en **HTTPS** avec un certificat valide. En HTTP, le jeu se lance mais se rabat sur les boutons tactiles.

## Étapes

1. Choisir l'URL : un sous-domaine (ex. `kart.<domaine>`) ou un chemin sur un domaine existant (ex. `<domaine>/kart/`).
2. Pour un sous-domaine : créer un enregistrement DNS A (et AAAA si IPv6) vers l'IP du VPS.
3. Copier le contenu de `site/` sur le serveur :
   ```bash
   sudo mkdir -p /var/www/kart-chrono
   sudo cp -r site/* /var/www/kart-chrono/
   sudo chown -R www-data:www-data /var/www/kart-chrono   # adapter l'utilisateur au serveur web en place
   ```
4. Configurer le serveur web déjà présent sur le VPS (ne pas en installer un second qui se battrait pour les ports 80/443). Exemples ci-dessous.
5. Vérifier (voir « Contrôle »).

### Si le VPS utilise Caddy

```caddy
kart.<domaine> {
    root * /var/www/kart-chrono
    encode gzip
    file_server
    @html path / /index.html
    header @html Cache-Control "no-cache"
}
```
Puis `sudo systemctl reload caddy`. Caddy obtient le certificat HTTPS tout seul.

### Si le VPS utilise nginx

```nginx
server {
    listen 80;
    server_name kart.<domaine>;
    root /var/www/kart-chrono;
    index index.html;

    location = /index.html { add_header Cache-Control "no-cache"; }
    location / { try_files $uri $uri/ =404; }

    types { application/manifest+json webmanifest; model/gltf-binary glb; }
    gzip on;
    gzip_types text/html application/javascript text/css application/manifest+json model/gltf-binary;
}
```
Puis `sudo nginx -t && sudo systemctl reload nginx`, et le certificat : `sudo certbot --nginx -d kart.<domaine>`.

Variante sous-dossier d'un site nginx existant : `location /kart/ { alias /var/www/kart-chrono/; }` dans le bloc `server` HTTPS déjà en place.

### Si le VPS tourne avec Docker et un reverse proxy (Traefik, Nginx Proxy Manager…)

Servir `site/` avec une image statique, par exemple :
```bash
docker run -d --name kart-chrono --restart unless-stopped \
  -v /var/www/kart-chrono:/usr/share/nginx/html:ro nginx:alpine
```
puis brancher le conteneur sur le reverse proxy existant avec le domaine choisi et un certificat Let's Encrypt.

## Contrôle

```bash
curl -sI https://kart.<domaine>/                      # 200, content-type text/html
curl -sI https://kart.<domaine>/vendor/three.min.js   # 200, application/javascript
curl -sI https://kart.<domaine>/manifest.webmanifest  # 200
curl -sI https://kart.<domaine>/models/pilote.glb     # 200 (si 404 : le dossier models/ n'a pas été copié)
```
Le certificat doit être valide (pas d'avertissement dans Safari).

## Mise à jour du jeu

Remplacer **tout** le contenu de `/var/www/kart-chrono/` par celui du dossier `site/` du nouveau zip (y compris les nouveaux dossiers `models/` et `vendor/`). Si `models/` manque, le jeu fonctionne quand même mais affiche un pilote simplifié. L'ancien fichier `models/kart.glb` n'est plus utilisé et peut être supprimé. `index.html` est servi en `no-cache`, donc le téléphone récupère la nouvelle version au prochain chargement. Les records déjà enregistrés sur le téléphone sont conservés tant que l'URL ne change pas.

## Côté iPhone (pour William)

1. Ouvrir l'URL dans **Safari**, téléphone à l'horizontale.
2. Partager → « Sur l'écran d'accueil » pour l'avoir en plein écran comme une app.
3. Au premier départ, accepter l'accès « Mouvement et orientation ».

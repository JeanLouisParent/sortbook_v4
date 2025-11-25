# Sortbook - composition Dockerisée

Ce dépôt fournit un scaffolding Docker Compose pour lancer :
- `traefik` (reverse-proxy)
- un conteneur `sqlite` (simple service qui assure la présence d'un fichier `/data/sqlite.db`)
- `n8n` (workflow)
- `flowise` (visual flow builder)
- `windmill` (automation)

Caractéristiques principales :
- Tous les conteneurs montent `/data` (chemin racine du système hôte) et montent également la variable `EPUB_ROOT` définie dans `.env` en `/epub_root`.
- Les workflows `n8n` et `windmill` peuvent être versionnés dans le dossier `workflow/`.
- Traefik sert les services via des règles basées sur le chemin :
  - `http://<IP-DU-SERVEUR>/n8n` → n8n
  - `http://<IP-DU-SERVEUR>/flowise` → Flowise
  - `http://<IP-DU-SERVEUR>/windmill` → Windmill

Fichiers clés :
- `docker-compose.yml` : orchestration
- `traefik/traefik.yml` : config statique Traefik
- `services/sqlite/Dockerfile` : conteneur SQLite minimal
- `workflow/` : emplacement pour stocker les workflows en texte

Variables d'environnement (fichier `.env`) :
- `EPUB_ROOT` : dossier racine des EPUBs (monté dans chaque conteneur)
- `EPUB_DEST` : dossier de sortie (utilisé par vos scripts internes)
- `LOG_DIR` : dossier de logs à l'intérieur des conteneurs
- `OLLAMA_URL` : URL du serveur Ollama (tourne sur une autre machine)

Lancement

1. Créez `/data` à la racine si ce n'est pas déjà fait et donnez les permissions nécessaires (sur macOS parfois sudo sera requis) :

```bash
sudo mkdir -p /data
sudo chown $USER:staff /data
```

2. Vérifiez et adaptez `.env` si nécessaire (chemins Windows fournis par défaut comme exemple).

3. Démarrer les services :

```bash
docker compose up -d
```

4. Accéder depuis un autre poste de votre réseau local :
- Ouvrez `http://<IP-DU-SERVEUR>/n8n` ou `/flowise` ou `/windmill` selon le service.

Notes & conseils
- Les images `windmillio/windmill:latest` et `flowiseai/flowise:latest` sont référencées comme exemples ; si vos images officielles diffèrent, modifiez `docker-compose.yml`.
- Si vous préférez un routage par nom d'hôte (ex. `n8n.sortbook.local`), ajoutez des règles Traefik `Host()` et adaptez votre DNS / fichier `/etc/hosts` sur les clients.
- Les données persistantes sont liées sur `/data` (sur l'hôte) : évitez de committer ce dossier.

# Edoras local stack (Traefik + n8n + Windmill + Flowise)

Ce repo contient une stack Docker locale pour :
- exposer plusieurs services via Traefik ;
- partager une bibliothèque d'EPUB ;
- consommer une instance Ollama distante.

## Services

- Traefik (reverse proxy + dashboard)
  - Dashboard : `http://edoras.local:8080/dashboard/`
- n8n : `https://n8n.edoras.local`
- Windmill : `https://windmill.edoras.local`
- Flowise : `https://flowise.edoras.local`

Tous les services passent par Traefik en HTTPS avec un certificat autosigné.

## Pré-requis

- Docker / Docker Desktop
- Fichier `.env` rempli (voir plus bas)
- Accès réseau à l’instance Ollama distante :
  - IP : `192.168.1.49`
  - Port : `11434`

## Fichier `.env`

Variables principales utilisées par `docker-compose.yml` :

```env
DOMAIN_NAME=edoras.local
SUBDOMAIN=n8n
GENERIC_TIMEZONE=Europe/Paris

# Windmill
WM_IMAGE=ghcr.io/windmill-labs/windmill:latest
WINDMILL_DB_PASSWORD=changeme
DATABASE_URL=postgres://windmill:${WINDMILL_DB_PASSWORD}@windmill_db:5432/windmill

# Flowise
FLOWISE_PASSWORD=changeme
FLOWISE_SECRETKEY=supersecret

# Librairie EPUB (dossier physique sur la machine hôte)
EPUB_ROOT=G:\Livres bruts

# Ollama distant
OLLAMA_URL=http://192.168.1.49:11434
```

Adapte `EPUB_ROOT` et les mots de passe à ton environnement.

## Résolution DNS locale (fichier hosts)

Sur chaque machine cliente (Mac, Linux, Windows), tu dois faire pointer les sous-domaines vers l’IP de la machine Docker (par ex. `192.168.1.56`).

Exemple sur macOS / Linux (`/etc/hosts`) :

```bash
sudo sh -c "echo '\n# Edoras hosts\n192.168.1.56 edoras.local n8n.edoras.local flowise.edoras.local windmill.edoras.local' >> /etc/hosts"
```

Exemple sur Windows (`C:\Windows\System32\drivers\etc\hosts`, éditeur en admin) :

```text
192.168.1.56 edoras.local n8n.edoras.local flowise.edoras.local windmill.edoras.local
```

Remplace `192.168.1.56` par l’IP locale réelle de la machine qui exécute Docker.

## Volumes et données

Toutes les données persistantes sont stockées dans le dossier `./data` du repo :

- n8n : `./data/n8n`, `./data/local-files`
- Windmill :
  - DB Postgres : `./data/windmill/db`
  - logs / cache worker : `./data/windmill/worker_logs`, `./data/windmill/worker_cache`
- Flowise :
  - DB Mongo : `./data/flowise/db`

Ce dossier est ignoré par Git (voir `.gitignore`) pour éviter de committer des fichiers volumineux.

La bibliothèque d’EPUB (`EPUB_ROOT`) est montée en lecture seule dans tous les conteneurs sur `/epub`.

## Ollama

Les services n8n et Flowise reçoivent `OLLAMA_URL` en variable d’environnement.  
L’URL par défaut est :

- `http://192.168.1.49:11434`

Tu peux l’adapter dans `.env` si ton instance Ollama change d’IP ou de port.

## Traefik

Configuration principale :

- `traefik/traefik.yml` : entrypoints, provider file, dashboard.
- `traefik/dynamic.yml` : routers / services / TLS :
  - `n8n.edoras.local` → `sortbook_n8n:5678`
  - `windmill.edoras.local` → `sortbook_windmill_server:8000`
  - `flowise.edoras.local` → `sortbook_flowise:3000`
  - certificat autosigné `traefik/certs/edoras.local.{crt,key}` monté dans le conteneur.

## Démarrage

Depuis la racine du repo :

```bash
docker compose up -d
```

Puis, depuis une machine cliente avec le `hosts` configuré :

- n8n : `https://n8n.edoras.local`
- Windmill : `https://windmill.edoras.local`
- Flowise : `https://flowise.edoras.local`
- Dashboard Traefik : `http://edoras.local:8080/dashboard/`

## Dépannage rapide

- 404 sur un service :
  - vérifier que le conteneur est `Up` (`docker ps`)
  - vérifier la ligne `hosts` et l’IP
  - tester en local sur la machine Docker :
    - `curl -vk https://localhost -H "Host: n8n.edoras.local"`

- Problèmes de cache / HTTPS sur le navigateur :
  - tester en navigation privée ou avec un autre navigateur.
*** End Patch

## Scope

Ce fichier décrit comment travailler avec cette stack Docker locale (Traefik + n8n + Windmill + Flowise) dans ce repo.

Il s’applique à tout le dépôt.

## Structure générale

- `docker-compose.yml` : définition des services et des volumes.
- `traefik/` :
  - `traefik.yml` : config statique Traefik (entrypoints, provider file, dashboard).
  - `dynamic.yml` : config dynamique (routers / services / TLS).
  - `certs/` : certificat autosigné pour `*.edoras.local`.
- `data/` : données persistantes des services (ignoré par Git).
- `.env` : configuration de l’environnement (domaine, DB, Ollama, EPUB_ROOT, etc.).

## Principes

- Ne jamais committer :
  - `.env`
  - le contenu du dossier `data/`
- Toutes les données persistantes doivent aller sous `./data/...` (et non dans des volumes nommés anonymes).
- La bibliothèque d’EPUB doit être configurée via `EPUB_ROOT` dans `.env` et montée dans les conteneurs sur `/epub`.
- L’instance Ollama distante doit être référencée via `OLLAMA_URL` dans `.env`.

## Services et URLs

- Traefik dashboard :
  - `http://edoras.local:8080/dashboard/`
- n8n :
  - `https://n8n.edoras.local`
- Windmill :
  - `https://windmill.edoras.local`
- Flowise :
  - `https://flowise.edoras.local`

Le TLS est assuré par un certificat autosigné stocké dans `traefik/certs`.

## Fichier hosts

Pour développer / tester depuis d’autres machines du LAN, ajouter une ligne dans le fichier `hosts` de chaque machine cliente, en pointant vers l’IP de la machine Docker (par exemple `192.168.1.56`).

Exemples :

- macOS / Linux (`/etc/hosts`) :

  ```bash
  sudo sh -c "echo '\n# Edoras hosts\n192.168.1.56 edoras.local n8n.edoras.local flowise.edoras.local windmill.edoras.local' >> /etc/hosts"
  ```

- Windows (`C:\Windows\System32\drivers\etc\hosts`) :

  ```text
  192.168.1.56 edoras.local n8n.edoras.local flowise.edoras.local windmill.edoras.local
  ```

Remplacer `192.168.1.56` par l’IP réelle de la machine qui héberge Docker.

## Style et conventions

- Garder `docker-compose.yml` organisé par blocs :
  - Traefik
  - n8n
  - Windmill (db + server + worker)
  - Flowise (db + app)
- Ajouter des commentaires concis dans les fichiers YAML pour les nouvelles sections.
- Ne pas multiplier les fichiers Traefik : utiliser `traefik.yml` et `dynamic.yml` existants.

## Démarrage / arrêt

- Démarrer la stack :

  ```bash
  docker compose up -d
  ```

- Arrêter :

  ```bash
  docker compose down
  ```

- Réappliquer la config Traefik après changement de `dynamic.yml` si nécessaire :

  ```bash
  docker restart sortbook_traefik
  ```


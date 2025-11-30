# Table of Contents
- [Jellyfin Service](#jellyfin-service)
- [FileBrowser Service](#filebrowser-service)
- [SplitPro Service](#splitpro-service)
- [Stirling-PDF Service](#stirling-pdf-service)
- [Paperless-ngx Service](#paperless-ngx-service)

## Jellyfin Service
Jellyfin is an open-source media server software that allows you to organize, manage, and stream your media files such as movies, TV shows, music, and photos to various devices. It provides a user-friendly interface and supports a wide range of platforms, making it easy to access your media library from anywhere.

Docker Compose configuration for Jellyfin:
```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ="America/New_York"
      - JELLYFIN_PublishedServerUrl=http://xxx.xxx.xxx.xxx #optional
    volumes:
      - /mnt/somedisk/docker/jellyfin/config:/config
      - /mnt/somedisk/docker/jellyfin/tvshows:/data/tvshows
      - /mnt/somedisk/docker/jellyfin/movies:/data/movies
      - /mnt/somedisk/docker/jellyfin/music:/data/music
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional
    env_file:
      - stack.env
    restart: unless-stopped
```

Environment Variables:
- `PUID`: User ID for the container to run as (replace with your user ID).
- `PGID`: Group ID for the container to run as (replace with your group ID).

## FileBrowser Service
FileBrowser is a web-based file management tool that allows you to easily browse, upload, download, and manage files on your server through a user-friendly interface. It provides features such as user authentication, file sharing, and directory management.

Docker Compose configuration for FileBrowser:
```yaml
version: "3.8"

services:
  filebrowser:
    image: filebrowser/filebrowser:s6
    container_name: filebrowser
    ports:
      - "8080:80"
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
    volumes:
      - /yourlocation/data:/srv
      - /yourlocation:/database
      - /yourlocation:/config
    restart: unless-stopped
```

Environment Variables:
- `PUID`: User ID for the container to run as (replace with your user ID).
- `PGID`: Group ID for the container to run as (replace with your group ID).


## SplitPro Service
[SplitPro](https://splitpro.app/) is a web-based application, that is an alternative to the popular Splitwise app. It allows users to manage shared expenses and track who owes whom in a simple and organized manner. You can find the [source code here](https://github.com/oss-apps/split-pro).

Docker Compose configuration for SplitPro:
```yaml
name: split-pro-prod

services:
  postgres:
    image: postgres:16
    container_name: ${POSTGRES_CONTAINER_NAME:-splitpro-db}
    restart: always
    environment:
      - POSTGRES_USER=${POSTGRES_USER:?err}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:?err}
      - POSTGRES_DB=${POSTGRES_DB:?err}
      - POSTGRES_PORT=${POSTGRES_PORT:-5432}
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${POSTGRES_USER}']
      interval: 10s
      timeout: 5s
      retries: 5
    # ports:
    #   - "5432:5432"
    volumes:
      - /home/someone/docker/split/data:/var/lib/postgresql/data

  splitpro:
    image: ossapps/splitpro:latest
    container_name: splitpro
    restart: always
    ports:
      - ${PORT:-3000}:${PORT:-3000}
    environment:
      - PORT=${PORT:-3000}
      - HOSTNAME=0.0.0.0 # Uncomment this, to have the server listen on all network interfaces. Needed when you want to connect to split-pro via docker internal networks, e.g. for reverse proxy setups.
      - DATABASE_URL=${DATABASE_URL:?err}
      - NEXTAUTH_URL=${NEXTAUTH_URL:?err}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET:?err}
      - ENABLE_SENDING_INVITES=${ENABLE_SENDING_INVITES:?err}
      - FROM_EMAIL=${FROM_EMAIL}
      - EMAIL_SERVER_HOST=${EMAIL_SERVER_HOST}
      - EMAIL_SERVER_PORT=${EMAIL_SERVER_PORT}
      - EMAIL_SERVER_USER=${EMAIL_SERVER_USER}
      - EMAIL_SERVER_PASSWORD=${EMAIL_SERVER_PASSWORD}
      - GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
      - GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}
      - AUTHENTIK_ID=${AUTHENTIK_ID}
      - AUTHENTIK_SECRET=${AUTHENTIK_SECRET}
      - AUTHENTIK_ISSUER=${AUTHENTIK_ISSUER}
      - R2_ACCESS_KEY=${R2_ACCESS_KEY}
      - R2_SECRET_KEY=${R2_SECRET_KEY}
      - R2_BUCKET=${R2_BUCKET}
      - R2_URL=${R2_URL}
      - R2_PUBLIC_URL=${R2_PUBLIC_URL}
      - WEB_PUSH_PRIVATE_KEY=${WEB_PUSH_PRIVATE_KEY}
      - WEB_PUSH_PUBLIC_KEY=${WEB_PUSH_PUBLIC_KEY}
      - WEB_PUSH_EMAIL=${WEB_PUSH_EMAIL}
      - FEEDBACK_EMAIL=${FEEDBACK_EMAIL}
      - DISCORD_WEBHOOK_URL=${DISCORD_WEBHOOK_URL}
      - DEFAULT_HOMEPAGE=${DEFAULT_HOMEPAGE}
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  database:
```

Configure the environment variables in a `stack.env` file or directly in your Portainer environment.
```ini
POSTGRES_CONTAINER_NAME=splitpro-db
POSTGRES_USER=************
POSTGRES_PASSWORD=**********
POSTGRES_DB=********
POSTGRES_PORT=5432
DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_CONTAINER_NAME}:${POSTGRES_PORT}/${POSTGRES_DB}
PORT=3000
DEFAULT_HOMEPAGE=/home
ENABLE_SENDING_INVITES=false
DISABLE_EMAIL_SIGNUP=false
AUTHENTIK_ID=*****************************
AUTHENTIK_SECRET=*****************************
AUTHENTIK_ISSUER=<https://authentication.domain.in/application/o/provider>
NEXTAUTH_URL=<subdomainname.yourdomain.com>
NEXTAUTH_SECRET=*****************************
```

**NOTE:** You can configure authentik as OAUTH provider for SplitPro with Two factor authentication.

## Stirling-PDF Service

Stirling-PDF is a web-based application that allows users to convert, merge, split, and manipulate PDF files easily through a user-friendly interface. It provides various tools for managing PDF documents efficiently.

Docker Compose configuration for Stirling-PDF:
```yaml
version: '3.3'
services:
  stirling-pdf:
    image: stirlingtools/stirling-pdf:latest
    container_name: stirling-pdf
    ports:
      - '8080:8080'
    volumes:
      - /home/somedrive/docker/stirlingpdf/stirling-data:/configs
    restart: unless-stopped
```

## Paperless-ngx Service

Paperless-ngx is a document management system that allows users to digitize, organize, and manage their paper documents efficiently. It provides features such as OCR (Optical Character Recognition), tagging, and searching for easy retrieval of documents.

Docker Compose configuration for Paperless-ngx:
```yaml
# Docker Compose file for running paperless from the Docker Hub.
# This file contains everything paperless needs to run.
# Paperless supports amd64, arm and arm64 hardware.
#
# All compose files of paperless configure paperless in the following way:
#
# - Paperless is (re)started on system boot, if it was running before shutdown.
# - Docker volumes for storing data are managed by Docker.
# - Folders for importing and exporting files are created in the same directory
#   as this file and mounted to the correct folders inside the container.
# - Paperless listens on port 8010.
#
# In addition to that, this Docker Compose file adds the following optional
# configurations:
#
# - Instead of SQLite (default), PostgreSQL is used as the database server.
#
# To install and update paperless with this file, do the following:
#
# - Open portainer Stacks list and click 'Add stack'
# - Paste the contents of this file and assign a name, e.g. 'paperless'
# - Upload 'docker-compose.env' by clicking on 'Load variables from .env file'
# - Modify the environment variables as needed
# - Click 'Deploy the stack' and wait for it to be deployed
#
# For more extensive installation and update instructions, refer to the
# documentation.
services:
  broker:
    image: docker.io/library/redis:8
    restart: unless-stopped
    volumes:
      - /home/navin/docker/paperless/redisdata:/data
  db:
    image: docker.io/library/postgres:18
    restart: unless-stopped
    volumes:
      - /home/navin/docker/paperless/pgdata:/var/lib/postgresql
    environment:
      POSTGRES_DB: paperless
      POSTGRES_USER: paperless
      POSTGRES_PASSWORD: paperless
  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    restart: unless-stopped
    depends_on:
      - db
      - broker
    ports:
      - "8000:8000"
    volumes:
      - /home/somedrive/docker/paperless/data:/usr/src/paperless/data
      - /home/somedrive/docker/paperless/media:/usr/src/paperless/media
      - /home/somedrive/docker/paperless/export:/usr/src/paperless/export
      - /home/somedrive/docker/paperless/consume:/usr/src/paperless/consume
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBHOST: db
    env_file:
      - stack.env
volumes:
  data:
  media:
  pgdata:
  redisdata:
  ```

Environment Variables:
- COMPOSE_PROJECT_NAME=paperless
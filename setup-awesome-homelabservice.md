# Table of Contents
- [Jellyfin Service](#jellyfin-service)
- [FileBrowser Service](#filebrowser-service)

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


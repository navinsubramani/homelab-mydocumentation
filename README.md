# Home Data Center
This repository contains information about how to set up a home data center to sync your data from your phone to the home data center

# Table of contents

- [Sync Photos and Videos from Phone to Home Cloud (Locally)](#sync-photos-from-phone-to-cloud)
  - [Hardware Needed](#1-hardware-needed)
  - [Setup Procedure](#1-procedure)
    - [Installing Ubuntu OS on Raspberry Pi and bringup](#installing-ubuntu)
    - [Setup the Hard Disk on Pi](#setup-harddisk)
    - [Install Docker on the Raspberry Pi](#installing-docker)
    - [Install Portainer to manage all Containers](#installing-porter)
    - [Install Immich and add to Portainer](#installing-immich)
    - [Setup Immich on Mobile and Server](#setup-immich)


# Sync Photos and Videos from Phone to Home Cloud <a name="sync-photos-from-phone-to-cloud"></a>

## Hardware Needed <a name="1-hardware-needed"></a>
1. Raspberry Pi 5 (8GB RAM, 16GB SD card, and its kit)
2. Hard Disk 4TB Seagate [on amazon](https://www.amazon.com/dp/B094R1YV68?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)

## Procedure <a name="1-procedure"></a>
### Installing Ubuntu OS on Raspberry Pi and bringup <a name="installing-ubuntu"></a>
1. Download the `latest Ubuntu OS` for Pi from [here](https://ubuntu.com/download/raspberry-pi). Review if this is supported for the model of pi that you have.
2. Install OS using the `PI Manager` [Raspberry Pi Manager](https://www.raspberrypi.com/software/) on the SD card.
3. Mount and set up the PI. Please make sure to connect the PI to your home network through WIFI or LAN.

### Setup the Hard Disk on Pi <a name="setup-harddisk"></a>
1. Once the Hard Disk is connected to the Pi, Open `Disks` and you will notice the SD Card reader and Hard Disk.
2. Delete any existing Volume and create a new volume.
3. On the Volume --> Click on `Additional Partition Options` -->  and click `Edit Mount Options`
```
User Session Defaults = False
Mount Options: Mount at system startup = false
Mount Options: Show in user interface = false
Display Name = "saysomename" # Provide a display name
Mount Point = "/mnt/saysomename"
```
4. Start the Volumes in `Disks`
5. Under contents, check if the `mounted at path` is valid based on your above configuration, and it is accessible.

### Install Docker on the Raspberry Pi <a name="installing-docker"></a>

1. Follow the Instructions from [Digital Ocean Guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04) on how to install Docker for the specific version of Ubuntu.

_NOTE:_ All services are installed as Docker images/containers, so Docker is used to host different services on the Pi.

_NOTE:_  Install Docker engine, add current user to Docker so Docker functions work without sudo, test downloading images, and creating containers.

_NOTE:_  On Docker permission denied error, please follow one of the methods to fix it. [Fix for Docker Permission Denied](https://phoenixnap.com/kb/docker-permission-denied)

### Install Portainer to manage all Containers <a name="installing-porter"></a>
1. Read about [Portainer](https://www.portainer.io/). Portainer is a container management solution that manages containers from different environments from a single place through the browser. Using this, we can control the Pi services without logging into Pi. For this application, Portainer will manage only the local environment.
2. Install Portainer from the Hard Disk to separate the data from the Raspberry Pi: Go to `Hard Disk` --> Create a folder `docker/portainer`
3. Under this folder, create a file called `docker-compose.yaml` using the terminal opened in the same folder.

```
touch docker-compose.yaml
```

4. Edit the `docker-compose.yaml` file and copy and paste the below configuration. Edit only the commented sections below based on your configuration.

```
# docker-compose.yml 
version: '3'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    network_mode: host
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer-data:/data
  agent:
    container_name: portainer_agent
    restart: always
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    image: 'portainer/agent:latest'
    ports:
      - '9001:9001'
```

5. Once the `docker-compose.yaml` file is created, in the same terminal execute the below command.

```
docker compose up -d
```

6. This will pull the Docker images and create the container. Now you can access the `Portainer` from Pi using [localhost:9000](localhost:9000) or from other devices in the same network using `<pi ip address>:9000`.
7. Sign up for the Portainer and remember the credentials.
8. Under Portainer, you will notice the environment as `local`, pointing to the Docker in the local environment. Clicking on it, you can now view the 1 stack for `portainer`, a list of `images`, and `containers` created under it.

### Install Immich and add to Portainer <a name="installing-immich"></a>
1. Installing `Immich` for Photo Cloud can be done using their documentation from their [website](https://immich.app/docs/install/docker-compose). Alternatively, you can also use the below YAML file to set up Immich through our Portainer.
2.  Got to `Portainer` --> `local` environment --> `dashboard` --> `stack` --> `Add Stack`
3.  Provide a name, say `Immich`, choose the `Web Editor`, and paste the below yaml into the editor.

```
version: "3.8"

#
# WARNING: Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.
#

name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - '/mnt/saysomename/docker/immich/data/Personal:/mnt/media/Personal:ro' # Edit the location to the left of the ':' and point to your Personal folder on the hard disk where on-demand media files are stored
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - stack.env
    ports:
      - 2283:2283
    depends_on:
      - redis
      - database
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    volumes:
      - '/mnt/saysomename/docker/immich/config/model-cache:/cache' # Edit the location to the left of the ':' and point to your cache folder on the hard disk where immich-cache should be stored
    env_file:
      - stack.env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/redis:latest
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:739cdd626151ff1f796dc95a6591b55a714f341c737e27f045019ceabf8e8c52
    env_file:
      - stack.env
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    healthcheck:
      test: pg_isready --dbname='${DB_DATABASE_NAME}' || exit 1; Chksum="$$(psql --dbname='${DB_DATABASE_NAME}' --username='${DB_USERNAME}' --tuples-only --no-align --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM pg_stat_database')"; echo "checksum failure count is $$Chksum"; [ "$$Chksum" = '0' ] || exit 1
      interval: 5m
      start_interval: 30s
      start_period: 5m
    command: 
      [
        'postgres',
        '-c',
        'shared_preload_libraries=vectors.so',
        '-c',
        'search_path="$$user", public, vectors',
        '-c',
        'logging_collector=on',
        '-c',
        'max_wal_size=2GB',
        '-c',
        'shared_buffers=512MB',
        '-c',
        'wal_compression=on',
      ]
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    restart: always
```

4. Scroll down, and under the `Environment Variables` --> `Advanced Mode`, add the below variables.

```
UPLOAD_LOCATION="/mnt/saysomename/docker/immich/data/Uploads" # Point to the folder in the Hard Disk where the backup should be stored
IMMICH_VERSION=release
DB_PASSWORD="" # Mention the database password here
DB_HOSTNAME=immich_postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
REDIS_HOSTNAME=immich_redis
DB_DATA_LOCATION="/mnt/saysomename/docker/immich/config/db" # Point to the folder in the Hard Disk where the backup should be stored.
TZ="America/New_York"
```

5. Deploy the stack. This takes some time to download and install all the images and create a container.
6. Once done, you can access the Immich from local on [localhost:2283](localhost:2283) or from the network devices from `<pi ip address>:2283`.

### Setup Immich on Mobile and Server <a name="setup-immich"></a>

1. On Serer/Pi through the browser (mentioned above), open Immich and sign up. Please remember the credentials.
2. Once signed up, you can review the settings from `profile icon` --> `Administration` --> `Settings`.
3. You can add new users from `Administration` --> `Users`
4. On Mobile, you can install the `Immich mobile application` from App Store/scanning from their [site](https://immich.app/).
5. Once the app is installed and opened, you can enter the network URL with http into it to connect to `Immich Local Server`. With this, you can successfully start backing up the photos from your phone to your server through this application.


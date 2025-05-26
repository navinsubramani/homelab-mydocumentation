# Table of Contents
- [Install Immich on Server](#installing-immich)
- [Setup Immich App on Mobile](#setup-immich)
- [Setup Authentik for Immich (Optional)](#setup-authentik-for-immich)


# Install Immich and add to Portainer <a name="installing-immich"></a>
1. Installing `Immich` for Photo Cloud can be done using their documentation from their [website](https://immich.app/docs/install/docker-compose). Alternatively, you can also use the below YAML file to set up Immich through our Portainer.
2.  Got to `Portainer` --> `local` environment --> `dashboard` --> `stack` --> `Add Stack`
3.  Provide a name, say `Immich`, choose the `Web Editor`, and paste the below yaml into the editor.

```yaml
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

```ini
UPLOAD_LOCATION="/mnt/saysomename/docker/immich/data/Uploads" # Point to the folder in the Hard Disk where the backup should be stored
IMMICH_VERSION="release"
DB_PASSWORD="" # Mention the database password here
DB_HOSTNAME="immich_postgres"
DB_USERNAME="postgres"
DB_DATABASE_NAME="immich"
REDIS_HOSTNAME="immich_redis"
DB_DATA_LOCATION="/mnt/saysomename/docker/immich/config/db" # Point to the folder in the Hard Disk where the backup should be stored.
TZ="America/New_York"
```

5. Deploy the stack. This takes some time to download and install all the images and create a container.
6. Once done, you can access the Immich from local on [localhost:2283](localhost:2283) or from the network devices from `<pi ip address>:2283`.

# Setup Immich on Mobile and Server <a name="setup-immich"></a>

1. On Server/Pi through the browser (mentioned above), open Immich and sign up. Please remember the credentials.
2. Once signed up, you can review the settings from `profile icon` --> `Administration` --> `Settings`.
3. You can add new users from `Administration` --> `Users`
4. On Mobile, you can install the `Immich mobile application` from App Store/scanning from their [site](https://immich.app/).
5. Once the app is installed and opened, you can enter the network URL with http into it to connect to `Immich Local Server`. With this, you can successfully start backing up the photos from your phone to your server through this application.

# Setup Authentik for Immich (Optional)

Follow the Instruction from the [Setup Authentik](setup-authentik.md) to set the services up first and then follow this instruction to configure the Autehntik as a SSO for Immich.

1. Open the `Immich` server from the browser and log in.
2. Go to `Administration` --> `Settings` --> `Authentication`.
3. Under `OAuth`, enable the OAuth and provide the below details.
    - **Issuer URL**: `Copy the Issuer URL from Authentik Provider` 
    - **Client ID**: `authentik Client ID`
    - **Client Secret**: `client secret from authentik`
    - **Token Endpoint**: `client_secret_post`
    - **Scoper**: `openid email profile`
    - **Storage Label Claim**: `preferred_username`

Additionally, you also refer to the [Immich Authentik Documentation](https://docs.goauthentik.io/integrations/services/immich/) for more details.
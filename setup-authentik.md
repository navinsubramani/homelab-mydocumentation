# Table of Contents
- [Install Authentik Service on the server](#setup-authentik)
- [Configure Authentik, Application, Provider, Certifications](#configure-authentik)
    - [Configure Authentik](#configure-authentik)
    - [Create Application](#create-application)
    - [Create Provider](#create-provider)
    - [Create a Certificates](#create-a-certificates)



# Install Authentik Service on the server <a name="setup-authentik"></a>
1. `Authentik` is an open-source identity provider that can be used to manage user authentication for various services, including Immich. It provides a centralized authentication system that can be integrated with multiple applications.
2. To install Authentik, you can follow the official documentation from their [website](https://docs.goauthentik.io/docs/install-config/install/docker-compose). Alternatively, you can use the below YAML file to set up Authentik through Portainer.
3. Go to `Portainer` --> `local` environment --> `dashboard` --> `stack` --> `Add Stack`
4. Provide a name, say `Authentik`, choose the `Web Editor`, and paste the below YAML into the editor.

```yaml
name: authentik

services:
  postgresql:
    container_name: Authentik-Postgres
    image: docker.io/library/postgres:16-alpine
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      start_period: 20s
      interval: 30s
      retries: 5
      timeout: 5s
    volumes:
      - /mnt/somedrive/docker/authentik/postgres:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${PG_PASS:?database password required}
      POSTGRES_USER: ${PG_USER:-authentik}
      POSTGRES_DB: ${PG_DB:-authentik}
    env_file:
      - stack.env

  redis:
    container_name: Authentik-Redis
    image: docker.io/library/redis:alpine
    command: --save 60 1 --loglevel warning
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep PONG"]
      start_period: 20s
      interval: 30s
      retries: 5
      timeout: 3s
    volumes:
      - /mnt/somedrive/docker/authentik/redis:/data
  
  server:
    container_name: Authentik-Server
    image: ${AUTHENTIK_IMAGE:-ghcr.io/goauthentik/server}:${AUTHENTIK_TAG:-latest}
    restart: unless-stopped
    command: server
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
    volumes:
      - /mnt/somedrive/docker/authentik/authentik_media:/media
      - /mnt/somedrive/docker/authentik/authentik_custom_templates:/templates
    env_file:
      - stack.env
    ports:
      - "${COMPOSE_PORT_HTTP:-9080}:9000"
      - "${COMPOSE_PORT_HTTPS:-9043}:9443"
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
  
  worker:
    container_name: Authentik-Worker
    image: ${AUTHENTIK_IMAGE:-ghcr.io/goauthentik/server}:${AUTHENTIK_TAG:-latest}
    restart: unless-stopped
    command: worker
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
    # `user: root` and the docker socket volume are optional.
    # See more for the docker socket integration here:
    # https://goauthentik.io/docs/outposts/integrations/docker
    # Removing `user: root` also prevents the worker from fixing the permissions
    # on the mounted folders, so when removing this make sure the folders have the correct UID/GID
    # (1000:1000 by default)
    user: root
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/somedrive/docker/authentik/authentik_media:/media
      - /mnt/somedrive/docker/authentik/authentik_certs:/certs
      - /mnt/somedrive/docker/authentik/authentik_custom_templates:/templates
    env_file:
      - stack.env
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
 ```

5. Create a file named `stack.env` in the same directory as the YAML file and add the following environment variables:
```ini
PG_PASS="your_database_password"
AUTHENTIK_SECRET_KEY="your_secret_key"
AUTHENTIK_ERROR_REPORTING_ENABLED="true"
```

_NOTE_: Before configuring the Autehntik service, ensure that this service is running properly and host it through Cloudflare domain so the Autehntik is configured to work over internet. You can refer to [setup clouflare, tunnel](setup-cloudflare-tunnel.md)

# Configure Authentik, Application, Provider, Certifications <a name="configure-authentik"></a>

## Configure Authentik
1. Go to `Portainer` --> `local` environment --> `dashboard` --> `stacks` and click on the `Authentik` stack you just created.
2. Click on the `Open in Browser` button to access the Authentik web interface.
3. Log in using the default credentials:
   - Username: `admin`
   - Password: `password`
4. After logging in, you will be prompted to change the password. Follow the instructions to set a new password.
5. Once logged in, you can configure Authentik by creating applications, providers, and certifications as needed.

## Create Application
6. To create an application for Immich, go to `Applications` and click on `Create` (instead of `Create Application with Provider`).
7. Fill in the application details:
   - Name: `your-home-lab-name`
   - Slug: `your-home-lab-slug-name`
   - Provider: Select the provider that you will be creating next. You can ignore and select it later as well.
8. Click on `Save` to create the application.

## Create Provider
9. Next, go to `Providers` and click on `Create` to create a new provider.
10. Fill in the provider details:
    - Name: `your-provider-name`
    - Authorization Flow: `Implicit`
    - Redirect URI: Select the Regex option and enter the following: `https?://([a-z0-9]+[.])*domainname[.]in.*`
    - Redirect URI: You can also create a explicit URI using the Strict option. For redirecting the URI to Immich mobile app, you can use the URI based in the Immich documentation from [here](setup-immich.md)
    - Signing Key: Select the key you will be creating next.
    - Protocol Scopes: Select the `email`, `offline_access`, `openid`, and `profile` scopes.
    - Subject Mode: Select `Email`.
11. Click on `Save` to create the provider.
12. Note the `Client ID` and `Client Secret` and `Issuer URL`. You will need these values when you configure the sevice like Immich.

## Create a Certificates
13. Go to `Certificates` and click on `Generate` to create a new certificate to generate the SSL certificate.
14. Use the generated certificate in the `provider` you created earlier.

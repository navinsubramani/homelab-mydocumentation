# Table of Contents
- [Install Portainer to manage all Containers](#installing-porter)
- [Configure Portainer with OAuth](#configure-portainer-oauth)


# Install Portainer to manage all Containers <a name="installing-porter"></a>
1. Read about [Portainer](https://www.portainer.io/). Portainer is a container management solution that manages containers from different environments from a single place through the browser. Using this, we can control the Pi services without logging into Pi. For this application, Portainer will manage only the local environment.
2. Install Portainer from the Hard Disk to separate the data from the Raspberry Pi: Go to `Hard Disk` --> Create a folder `docker/portainer`
3. Under this folder, create a file called `docker-compose.yaml` using the terminal opened in the same folder.

```bash
touch docker-compose.yaml
```

4. Edit the `docker-compose.yaml` file and copy and paste the below configuration. Edit only the commented sections below based on your configuration.

```yaml
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

```bash
docker compose up -d
```

6. This will pull the Docker images and create the container. Now you can access the `Portainer` from Pi using [localhost:9000](localhost:9000) or from other devices in the same network using `<pi ip address>:9000`.
7. Sign up for the Portainer and remember the credentials.
8. Under Portainer, you will notice the environment as `local`, pointing to the Docker in the local environment. Clicking on it, you can now view the 1 stack for `portainer`, a list of `images`, and `containers` created under it.


_NOTE:_ Portainer is used to manage all the containers and images created on the Pi. It provides a user-friendly interface to manage Docker containers, images, networks, and volumes.

# Configure Portainer with OAuth <a name="configure-portainer-oauth"></a>
1. Portainer supports OAuth authentication, which allows you to use external identity providers for user management. This is useful for managing access to your Portainer instance.
2. To configure OAuth in Portainer, you can setup a inhouse OAuth server using [Authentik](setup-authentik.md) and create applications and providers for Portainer.
3. Once you have Authentik set up, you can configure Portainer to use Authentik as an OAuth provider following steps from the [Authentik: Integrate with Portainer](https://docs.goauthentik.io/integrations/services/portainer/).
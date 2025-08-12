# Table of Contents
- [Setup Private Registry](#setup-private-registry)

# Setup Private Registry
A private Docker registry allows you to store and manage your own Docker images securely. This is useful for development, testing, and production environments where you want to control the images used in your containers.

To set up a private Docker registry, you can use the official Docker Registry image. Below is a sample `docker-compose.yml` file to run a private registry:

```yaml
version: "3.8"

services:
  registry-server:
    image: registry:2.8.2
    container_name: registry-server
    restart: always
    environment:
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin: '["http://xx.x.x.xxx:8090"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Methods: '["HEAD","GET","OPTIONS","DELETE"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Credentials: '["true"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Headers: '["Authorization","Accept","Cache-Control"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Expose-Headers: '["Docker-Content-Digest"]'
    volumes:
      - /mnt/somelocation/docker/registry/data:/var/lib/registry

  registry-ui:
    image: joxit/docker-registry-ui:latest
    container_name: registry-ui
    restart: always
    ports:
      - "8090:80"
    environment:
      SINGLE_REGISTRY: "true"
      REGISTRY_TITLE: "Registry UI"
      DELETE_IMAGES: "true"
      SHOW_CONTENT_DIGEST: "true"
      NGINX_PROXY_PASS_URL: "http://registry-server:5000"
      SHOW_CATALOG_NB_TAGS: "true"
      CATALOG_MIN_BRANCHES: "1"
      CATALOG_MAX_BRANCHES: "1"
      TAGLIST_PAGE_SIZE: "100"
      REGISTRY_SECURED: "false"
      CATALOG_ELEMENTS_LIMIT: "1000"
    depends_on:
      - registry-server
```

## Push and Pull Images to Your Private Registry
1. Tag your image to point to your private registry:
    ```bash
    docker tag your-image:tag xx.x.x.xxx:5000/your-image:tag
    ```

2. Push the image to your private registry:
    ```bash
    docker push xx.x.x.xxx:5000/your-image:tag
    ```

3. Pull the image from your private registry:
    ```bash
    docker pull xx.x.x.xxx:5000/your-image:tag
    ```

Make sure to replace `xx.x.x.xxx` with the actual IP address or domain name of your registry server. The registry UI will be accessible at `http://xx.x.x.xxx:8090`.

## Configure the Portainer to Use the Private Registry
To configure Portainer to use your private registry, follow these steps:
1. Log in to your Portainer instance.
2. Navigate to "Registries" in the left-hand menu.
3. Click on "Add registry".
4. Choose the "Custom registry" option.
5. Fill in the details:
    - Name: A name for your registry (e.g., "My Private Registry").
    - URL: The URL of your private registry (e.g., `http://xx.x.x.xxx:5000`).
    - Username and Password: If your registry requires authentication, provide the necessary credentials.

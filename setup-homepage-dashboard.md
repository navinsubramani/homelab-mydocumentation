# Table of Contents
- [Installation and Setup Guides for GetHomePage Service](#installation-and-setup-guides-for-gethomepage-service)
- [Connecting to Configuration File via VS Code](#connecting-to-configuration-file-via-vs-code)
- [Edit the `services.yaml`, `widgets.yaml` and `settings.yaml` files to customize your dashboard](#edit-the-servicesyaml-widgetsyaml-and-settingsyaml-files-to-customize-your-dashboard)
  - [settings.yaml](#settingsyaml)
  - [widgets.yaml](#widgetsyaml)
  - [services.yaml](#servicesyaml)

# Installation and Setup Guides for GetHomePage Service:
1. Go to `portainer` -> `Stacks` -> `Add Stack`. Name the stack `homepage`.
2. Copy and paste the following code in the `Web editor` section:
```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    environment:
      HOMEPAGE_ALLOWED_HOSTS: homepage.example.com # required, may need port. See gethomepage.dev/installation/#homepage_allowed_hosts
      PUID: ${PUID} # optional, your user id
      PGID: ${PGID} # optional, your group id
    ports:
      - 3000:3000
    volumes:
      - /home/somedisk/docker/homepage/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock:ro # optional, for docker integrations
    env_file:
      - stack.env
    restart: unless-stopped
```
3. Ensure to replace the value of `HOMEPAGE_ALLOWED_HOSTS` with your own domain or IP address.
4. Create a file named `stack.env` in the same directory as your `docker-compose.yml` file and add the following environment variables:
```ini
PUID=1000
PGID=1000
```
5. Click on `Deploy the stack` button to deploy the stack.


# Connecting to Configuration File via VS Code:
1. Open VS Code.
2. Install the "Remote - SSH" extension if you haven't already.
3. Click on the `Remote Explorer` icon in the Activity Bar on the side of the window.
4. Click on the `+` icon to add a new SSH target.
5. Enter the SSH connection details for your server (e.g., `user@hostname` or `user@IP_address`).
6. Click on the `Connect` button to connect to your server.
7. Once connected, navigate to the directory where your `docker-compose.yml` file is located.


# Edit the `services.yaml`, `widgets.yaml` and `settings.yaml` files to customize your dashboard:

1. Customize the `settings.yaml` file to set your preferred theme (light or dark) and other settings using the reference from [here](https://gethomepage.dev/configs/settings/).

2. Edit the `services.yaml` file to add, remove, or modify the services displayed on your dashboard using the reference from [here](https://gethomepage.dev/configs/services/).

3. Modify the `widgets.yaml` file to customize the widgets on your dashboard using the reference from [here](https://gethomepage.dev/widgets/).

4. To abstract the sensitive information from the yaml files, you can use the environment variables defined in the `stack.env` files. One important rule for `gethomepage` to load the environment variables is that the variable names must be in all uppercase letters and start with `HOMEPAGE_VAR_`. For example, if you have a variable named `HOMEPAGE_VAR_URL` in your `stack.env` file, you can reference it in your `services.yaml` file like this:

```yaml
- Services:
    PairDrop:
        icon: si-wikiquote-#006699
        href: {{HOMEPAGE_VAR_URL}}
```

## settings.yaml

```yaml
---
# For configuration options and examples, please see:
# https://gethomepage.dev/configs/settings/

title: Boring Home Lab
description: This is my homepage for all the awsome services I run at home.
background:
    image: https://images.unsplash.com/photo-1469474968028-56623f02e42e?q=80&w=2674&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
    blur: sm # sm, "", md, xl... see https://tailwindcss.com/docs/backdrop-blur
    saturate: 70 # 0, 50, 100... see https://tailwindcss.com/docs/backdrop-saturate
    brightness: 70 # 0, 50, 75... see https://tailwindcss.com/docs/backdrop-brightness
    opacity: 50 # 0-100

layout:
  - Hardwares:
      style: column
  
  - Infrastructure:
      style: column

  - Security:
      style: column

  - Applications-Media:
      style: column

  - Applications-Productivity:
      style: column

  - Applications-Development:
      style: column
  


providers:
  openweathermap: openweathermapapikey
  weatherapi: weatherapiapikey
```

## widgets.yaml
```yaml
---
# For configuration options and examples, please see:
# https://gethomepage.dev/configs/info-widgets/

- logo:
    href: https://boringengineer.com
    target: _blank # Optional, can be set in settings

- resources:
    cpu: true
    memory: true
    disk: /

- search:
    provider: google
    target: _blank
```

## services.yaml

_NOTE:_ I have NOT added my configuration files here yet, as I am still cleaning up the configuration so the data looks clean without revealing any of my IP address or domain.
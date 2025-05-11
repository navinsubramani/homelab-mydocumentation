# Home Data Center
This repository contains information about how to set up a home data center to sync your data from your phone to the home data center

# Table of contents
- [Sync Photos and Videos from Phone to Home Cloud](#sync-photos-from-phone-to-cloud)
  - [Hardware Needed](1-hardware-needed)
  - [Setup Procedure](1-procedure)
    - [Installing Ubuntu OS on Raspberry Pi and bringup](installing-ubuntu)
    - [Setup the Hard Disk on Pi](setup-harddisk)
    - [Install Docker on the Raspberry Pi](installing-docker)
    - [Install Portainer to manage all Containers](installing-porter)

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
8. Under Portainer, you will notice the environment as `local` pointing to the docker in the local environment. Clicking on it, you can now view the 1 stack for `portainer`, a list of `images`, and `containers` created under it.

### Add a Stack for Immich
1. 

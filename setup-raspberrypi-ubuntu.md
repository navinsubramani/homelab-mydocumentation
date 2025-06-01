# Table of Contents
- [Hardware Needed](#1-hardware-needed)
- [Installing Ubuntu OS on Raspberry Pi and bringup](#installing-ubuntu)
- [Setup the Hard Disk on Pi](#setup-harddisk)
- [Install Docker on the Raspberry Pi](#installing-docker)
- [Install Ruskdesk on Raspberry Pi](#setup-ruskdesk)
- [SSH into the pi from remote computer](#ssh-into-the-pi-from-remote-computer)
   - [Install htop to monitor the system](#install-htop-to-monitor-the-system)
   - [Identify the PUID and PGID](#identify-the-puid-and-pgid)


# Hardware Needed <a name="1-hardware-needed"></a>
1. Raspberry Pi 5 (8GB RAM, 16GB SD card, and its kit)
2. Hard Disk 4TB Seagate [on amazon](https://www.amazon.com/dp/B094R1YV68?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)


# Installing Ubuntu OS on Raspberry Pi and bringup <a name="installing-ubuntu"></a>
1. Download the `latest Ubuntu OS` for Pi from [here](https://ubuntu.com/download/raspberry-pi). Review if this is supported for the model of pi that you have.
2. Install OS using the `PI Manager` [Raspberry Pi Manager](https://www.raspberrypi.com/software/) on the SD card.
3. Mount and set up the PI. Please make sure to connect the PI to your home network through WIFI or LAN.

# Setup the Hard Disk on Pi <a name="setup-harddisk"></a>
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

# Install Docker on the Raspberry Pi <a name="installing-docker"></a>

1. Follow the Instructions from [Digital Ocean Guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04) on how to install Docker for the specific version of Ubuntu.

_NOTE:_ All services are installed as Docker images/containers, so Docker is used to host different services on the Pi.

_NOTE:_  Install Docker engine, add current user to Docker so Docker functions work without sudo, test downloading images, and creating containers.

_NOTE:_  On Docker permission denied error, please follow one of the methods to fix it. [Fix for Docker Permission Denied](https://phoenixnap.com/kb/docker-permission-denied)

# Install Ruskdesk on Raspberry Pi <a name="setup-ruskdesk"></a>
1. Follow the instructions from [Ruskdesk](https://rustdesk.com/)

_NOTE:_ Ruskdesk is a remote desktop solution that allows you to access your Pi from anywhere. It is an open-source alternative to TeamViewer and AnyDesk.


# SSH into the pi from remote computer
1. Open a terminal on your remote computer.
2. Use the following command to SSH into the Pi:
   ```bash
   ssh username@ip_address_of_pi
   ```
   Replace `username` with your Pi's username and `ip_address_of_pi` with the actual IP address of your Raspberry Pi.
3. If prompted, enter the password for the Pi.
4. Once logged in, you can run commands on the Pi as if you were using it directly.

## Install htop to monitor the system
1. Install `htop` to monitor the system performance.
   ```bash
   sudo apt install htop
   ```
2. Run `htop` to monitor the system performance.
   ```bash
    htop
    ```

## Identify the PUID and PGID
1. To find the PUID and PGID of the current user, run the following command:
   ```bash
   id <username>
   ``` 
   Replace `<username>` with your actual username.
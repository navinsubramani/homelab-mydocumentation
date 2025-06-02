# Table of Contents
- [Hardware Needed](#hardware-needed)
- [Installing Proxmox on the Computer](#installing-proxmox-on-the-computer)
- [Creating a Virtual Machine on your Proxmox Server](#creating-a-virtual-machine-on-your-proxmox-server)
  - [Setup Docker on Proxmox VM](#setup-docker-on-proxmox-vm)


# Hardware Needed
- Computer with minimum 16GB RAM and 4 CPU cores (as there should be enough RAM and CPU cores to run mulitple VMs)
- Pendrive with minimum 8GB capacity (to install Proxmox)
- Another computer to setup the pendrive and install Proxmox on the first computer

# Installing Proxmox on the Computer

_NOTE:_ I followed this step-by-step instruction to install Proxmox on my computer: [Proxmox Installation Guide](https://phoenixnap.com/kb/install-proxmox).

1. Follow the link above,
    - Download the Proxmox ISO file from the official website.
    - Create a bootable USB drive using the Proxmox ISO file.
    - Boot your computer from the USB drive.
    - Follow the installation wizard to install Proxmox on your computer.
    - Once the installation is complete, remove the USB drive and reboot your computer.
    - Access the Proxmox web interface by entering the IP address of your Proxmox server in a web browser.
    - Log in using the default username `root` and the password you set during installation.
    - Configure the Proxmox web interface as per your requirements.
    
# Creating a Virtual Machine on your Proxmox Server

_NOTE:_ Once Proxmox is installed, the creation of a VM with Ubuntu is similar to installing Ubuntu on a regular computer or pi. You can follow the steps from [Installing Ubuntu OS on Raspberry Pi and bringup](setup-raspberrypi-ubuntu.md#installing-ubuntu-os-on-raspberry-pi-and-bringup) to install Ubuntu on the VM.

1. Create a new virtual machine (VM) or container as needed.
2. Download the Ubuntu OS and store it in the `local` storage of Proxmox.
3. Install the required operating system on the VM or container.
4. Configure the VM or container settings as per your requirements.
5. Start the VM or container and verify that it is working correctly.

## Setup Docker on Proxmox VM
_NOTE:_ Once the VM is created, you can follow the steps from [Installing Docker on the Raspberry Pi](setup-raspberrypi-ubuntu.md#install-docker-on-the-raspberry-pi) to install Docker on the VM.

_NOTE:_ `local` storage in Proxmox is the storage that is available on the Proxmox server itself. You can use this storage to store the ISO files, VM images, and other data that you need for your Proxmox server. `local lvm` is a logical volume manager that allows you to create and manage logical volumes on your Proxmox server. You can use `local lvm` to create logical volumes for your VMs and containers, which can help you manage the storage more efficiently.
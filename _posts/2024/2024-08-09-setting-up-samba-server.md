---
title: Setting up SAMBA on Linux
description: Setting up samba server for file sharing
categories: [demo, tech]
date: 2024-08-09 13:45:00 +500
tags: [samba,setup,linux]     # TAG names should always be lowercase
pin: false
math: true
mermaid: true
image:
  path: https://ik.imagekit.io/yhmzg9hf6/server.webp?updatedAt=1723196390703 
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "Server image"
---


## Introduction

Setting up a Samba server on a Linux machine allows you to share files and directories with other devices on the same network, including Windows, Linux, and macOS systems. Here's how you can set up a Samba server:

##  1. Install Samba

First, you need to install Samba. The installation process may vary slightly depending on your Linux distribution.

### For Ubuntu/Debian:

```bash
sudo apt update
sudo apt install samba
```

### For CentOS/RHEL:

```bash
sudo yum install samba samba-client samba-common
```

### For Fedora:

```bash
sudo dnf install samba samba-client samba-common
```

## 2. Create a Shared Directory

Create a directory that you want to share with other users.

```bash
sudo mkdir -p /srv/samba/shared
```
Set the appropriate permissions on this directory:

```bash
sudo chmod 2775 /srv/samba/shared
sudo chown nobody:nogroup /srv/samba/shared
```
Alternatively, you can specify a different owner and group depending on your requirements.

## 3. Configure Samba

Edit the Samba configuration file, typically located at `/etc/samba/smb.conf`.

```bash
sudo nano /etc/samba/smb.conf
```
Add a section at the bottom of the file to define the shared directory:

```bash
[shared]
   path = /srv/samba/shared
   browseable = yes
   read only = no
   guest ok = yes
```
* [shared]: The name that will appear in the network for the share.
* path: The path to the directory you want to share.
* browseable: Whether the share should be visible in file managers.
* read only: Set to no to allow writing to the shared folder.
* guest ok: Set to yes to allow guest access (no password required).

## 4. Set Up Samba User Accounts

If you want to restrict access to the shared directory, you'll need to add Samba user accounts.

### Create a new system user (if needed):

```bash
sudo adduser <username>
```

### Add the user to Samba:

```bash
sudo smbpasswd -a <username>
```

You'll be prompted to enter and confirm a password.

## 5. Restart the Samba Service

After making changes to the configuration, restart the Samba service to apply them.

### For systemd-based distributions (e.g., Ubuntu, Fedora):

```bash
sudo systemctl restart smbd
```
### For SysVinit-based distributions:

```bash
sudo service smbd restart
```

## 7. Access the Samba Share

Now that the Samba server is set up, you can access the shared directory from other machines on the network.

### From Windows:

  * Open `File Explorer` and type `\\<server_ip>\shared` in the address bar.
  * Replace `<server_ip>` with the IP address of your Linux machine.

### From Linux:

  * You can use a file manager or `smbclient`:

    ```bash
    smbclient //server_ip/shared
    ```
### From macOS:

  * Open `Finder` and click on Go -> `Connect to Server`, then enter `smb://<server_ip>/shared`.

## 8. (Optional) Configure Automatic Mounting on Clients

For Linux clients, you can add an entry in the `/etc/fstab` file to mount the Samba share automatically at boot.

Example `fstab` entry:

```bash
//<server_ip>/shared /mnt/shared cifs username=<username>,password=<password>,iocharset=utf8 0 0
```
Replace `<server_ip>`, `<username>`, and `<password>` with your server's IP address, Samba username, and password.

Your Samba server should now be fully operational, allowing file sharing across different platforms on your network.


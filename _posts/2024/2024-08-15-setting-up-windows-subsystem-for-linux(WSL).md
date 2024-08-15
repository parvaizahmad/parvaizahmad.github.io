---
title: Setup Windows Subsystem for Linux (WSL)
description: Setup Windows Subsystem for Linux (WSL)
categories: [demo, tech]
date: 2024-08-15 09:18:00 +500
tags: [windows,WSL,setup,linux]     # TAG names should always be lowercase
pin: false
math: true
mermaid: true
image:
  path: https://ik.imagekit.io/yhmzg9hf6/wsl.png?updatedAt=1723695761681 
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "WSL Image"
---

## Setup Windows Subsystem for Linux (WSL)

### Main OS
- Windows 11
- Windows 10


### Hardware virtualization
- It should be enabled (check on `Task Manager` -> `Performance` -> `Virtualization: Enabled`)
- If it is not, I suggest enabling it on BIOS (on your CPU options enable `Intel VT-x` or `AMD-V`)
- Not sure if this is mandatory, but I always do this to ensure I can create standard virtual machines (with Virtualbox for example) it might have some performance impact on your WSL2 setup, just in case, enable it if you can

### Turn Windows features on or off
Go to windows feature and enable both:
- Virtual Machine Platform
- Windows Subsystem for Linux

### Restart machine
Linux folder will appear on file explorer

### Install / Update WSL
- Using Command Prompt (CMD) run the following several times until the latest version is installed
```bash
wsl --update
```

### Check available Linux distros
```bash
wsl --list --online
```

### Choose and install Linux distro

```bash
wsl --install -d Ubuntu-22.04
```
- Define UNIX username
- Define password

### Uninstall distribution
- If you eventually need to uninstall a Linux distribution from WSL, go to CMD
- Get the distribution name
```bash
wsl --list
```
- Unregister the distribution, **BE CAREFUL**, everything will be deleted, backup your data if needed
```bash
wsl --unregister Ubuntu-22.04
```
- Then go to "Add or remove programs", search for the distribution name and uninstall it

### Install terminal

From Microsoft Store install the official Terminal app by Microsoft Corporation, if you do not have it yet

### Default terminal
Make Linux distro the default terminal on Terminal app settings

### Update and Upgrade Linux distro
```bash
sudo apt update
sudo apt upgrade
```

### Tip to open Windows file explorer
- From Linux terminal, run the following command to open Windows file explorer in the current Linux directory
- Useful to transfer files etc.
```bash
explorer.exe .
```

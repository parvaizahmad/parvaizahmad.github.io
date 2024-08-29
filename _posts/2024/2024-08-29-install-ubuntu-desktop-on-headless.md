---
title: Install Ubuntu Desktop
description: We can install ubuntu desktop on cli based systems, vps or wsl
categories: [demo, tech]
date: 2024-08-29 16:08:00 +500
tags: [ubuntu]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
image:
  path: https://ik.imagekit.io/yhmzg9hf6/xrdp.png?updatedAt=1724929314828
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "xrdp"
---
## Headless System

A headless system refers to a computer or device that operates without a graphical user interface (GUI) or display. In other words, it doesn't have a monitor, keyboard, or mouse connected to it. Instead, it is typically controlled remotely through a network connection.
In the context of Ubuntu, a headless system can be a server or a device that doesn't require a graphical interface. It is commonly used for tasks such as web servers, file servers, IoT devices, or remote administration.
One example of a headless system in Ubuntu is a web server. You can install Ubuntu Server edition on a machine without a monitor, and then configure it to serve web pages. You can manage and administer the server remotely using SSH (Secure Shell) or other remote administration tools.
Another example is a Raspberry Pi running Ubuntu Server. Raspberry Pi is a popular single-board computer that can be used for various projects. By running Ubuntu Server on a Raspberry Pi, you can create a headless system for tasks like home automation, media streaming, or network monitoring.
In both cases, the absence of a graphical interface allows the system to be more lightweight and efficient, as it doesn't need to allocate resources for rendering graphics. Instead, it focuses on performing its intended tasks efficiently.
Do you have any specific questions or anything else you would like to know about headless systems in Ubuntu?

## Installing Desktop 

1. Update the server, install taskel then install ubuntu desktop:

    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    sudo apt install tasksel
    sudo tasksel install ubuntu-desktop
    ```

> On `WSL` after isntalling the `tasksel` run `sudo tasksel` and select a desktop environment, use `Spacebar` to select and `Tab` to select `ok` or `cancel`
{: .prompt-tip }


2. Install and enable remote desktop with xrdp

    ```bash
    sudo apt install xrdp -y
    sudo systemctl enable --now xrdp
    ```

3. Add the port 3389 to the iptables and save

    ```bash
    sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 3389 -j ACCEPT
    sudo netfilter-persistent save
    ```

4. Create a user for xdrp
    ```bash
    sudo adduser <userName>
    sudo usermod -G xrdp <userName>
    ```
5. Reboot machine/server

    ```bash
    sudo reboot
    ```

## Expected Issues

If you ran into an annoying popup issue “Authentication is required to create a color profile” when using Windows Remote Desktop. You can solve this by setting up a config for polkit.

1. Create a config file:

    ```bash
    sudo nano /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf
    ```

2. Paste in the following:

    ```bash
    /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf
    polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.color-manager.create-device" ||
    action.id == "org.freedesktop.color-manager.create-profile" ||
    action.id == "org.freedesktop.color-manager.delete-device" ||
    action.id == "org.freedesktop.color-manager.delete-profile" ||
    action.id == "org.freedesktop.color-manager.modify-device" ||
    action.id == "org.freedesktop.color-manager.modify-profile") &&
    subject.isInGroup("{users}")) {
    return polkit.Result.YES;
    }
    });
    ```

3. After saving the file, reboot the server:

    ```bash
    sudo reboot
    ```

## Connecting with Remote System

1. Find the ip address of the machine use `ifconfig` command and see the ip address

    ```bash
    ifconfig
    ```
2. Now open the `Remote Desktop Connection` utility on you windows machine and enter the ip address of machine in computer name section and click `Connect`

3. It will open up the desktop window, enter the username and password and click ok, Yaaaayy!! You are connected to Desktop environmet!


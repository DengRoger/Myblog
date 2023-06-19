---
title: Docker on porxmox
date: 2023-06-19 09:11:54
categories:
    - System
tags:
    - Docker
    - proxmox
---

# Installing Docker and Portainer to use With the Edge Agent on proxmox 
<!-- more -->

## 1. Install docker on proxmox VM

1. download turnkey-core 17.1-1 in proxmox templates 
![](https://i.imgur.com/DDa130J.png)

2. create LXC with turnkey-core 17.1-1 
3. get in to options -> Features to change options (keyctl , Nesting) to enable ![](https://i.imgur.com/YlkpZYZ.png)

4. after installed the system : 

    ```bash
    apt-get update

    apt-get upgrade

    apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

    mkdir -m 0755 -p /etc/apt/keyrings

    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    apt-get update

    apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

5. check does docker working ? 
![](https://i.imgur.com/EDck6Zl.png)

## Installing Docker and Portainer to use With the Edge Agent

1. do what this website did -> https://thehomelab.wiki/books/docker/page/installing-docker-and-portainer-to-use-with-the-edge-agent

    check your ip by : ``` ip -a ```
    type : <Docker LXC ip>:<port> on your browser 
    ![](https://i.imgur.com/tSkhfPI.png)


- resource
    - https://docs.docker.com/engine/install/debian/ 
    - https://thehomelab.wiki/books/docker/page/installing-docker-and-portainer-to-use-with-the-edge-agent 
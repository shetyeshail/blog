---
published: false
---
## Introduction

I've been meaning to start self-hosting a lot of the services I use on a regular basis by myself, but haven't really gotten around actually doing it. Every few months I would read an article about how companies are using data from users who use their products and it would make me want to switch, but it was enough of a hassle that I never actually took the plunge. Until this weekend.

A few days ago Google announced that they were removing the unlimited storage feature of Google Photos. I actually use that feature _a ton_. It also got me thinking of all the great [products](https://killedbygoogle.com/) that Google has also shut down over the years. If they decided to shut down Google Photos, Drive, Mail, etc. I would be ruined as basically all my digital data is backed up or stored in Google's ecosystem. It has been since I first created my Google Account for Gmail back when I was around 10 or so (with my parents' permission since I was under the age of 13, of course, if anyone at Google is reading this...)

However, it works quite well, and everything works cohesively together in a way you can't really duplicate with a mishmash of different products from different companies spanning different ecosystems (Google, Apple, Microsoft, etc). Another reason I was reluctant to self-host.

On top of that, there's the reliability and security aspect of it. I know my access to my account or products isn't going to go down very often if I'm hosted with one of these companies, since they all have at least three 9s of uptime, and it is very unlikely that they get hacked or lose my data since they have tons of very talented programmers and engineers working on these products to ensure everything stays safe.

I haven't really solved the cohesiveness issue, and only time will tell for the reliability (but I do have snapshots and backups in place), but I am pretty sure I have the kinks ironed out mostly for the security on on my own self-hosted stack. I'm happy so far with what I have setup, but I will look into expanding into more tools once I know my current setup is stable.

## Overview

I started brainstorming what tools I normally use on a daily basis. I looked for free self-hosted alternatives to them that I could easily deploy on my own. I have wanted to learn Docker for a while now, and this seemed like a good way to practice, so I decided to run all these apps from container images that already exist on DockerHub.

I wanted a nice way to manage my Docker containers from the web without having to login to my server each time to stop or restart a container, so I decided to use Portainer for that.

I also used NextCloud for a Google Suite alternative (Drive, Photos, etc). Bitwarden_rs is used for the password manager as a LastPass replacement. I went with Bitwarden_rs instead of the official Bitwarden build since it includes all the premium features for free and uses less system resources.

Everything is also setup to work behind an Nginx reverse proxy so I don't have to host all these services on different VPSes each.

I started with a Ubuntu 20.04 server, assigned my DNS with the Cloudflare CDN that I use for everything on my site, and got to work.

## DNS Settings

Create an **A record** in wherever you manage your domain DNS settings. I use Cloudflare CDN for mine so I created A records for the subdomains `portainer`, `bw`, and `nc`. Make sure to put the IP address of your domain name in the IP address field.

## Ubuntu Server

First I spun up a new Ubuntu 20.04 server instance. I updated the list of packages and installed some prereq packages. Setting up unattended-upgrades allows the system to download and install system updates itself in the background.
```shell
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

Then I added the GPG key for the Docker repo:
```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Next I add the Docker APT repo:
```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Now I can update the package list again so now it has the packages in the new Docker repo we just added and then we can install the necessary packages:
```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose
```

Now you'll have to check that the Docker service started properly.
```shell
sudo systemctl status docker
```

This should return the output as something similar to this:
```
● docker.service - Docker Application Container Engine
	Loaded:loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
    Active: active (running) since Sat 2020-11-14 04:02:43 CET; 1 day 1h ago
	TriggeredBy: ● docker.socket
    Docs: https://docs.docker.com
    Main PID: 3456 (dockerd)
    Tasks: 57
    Memory: 417.1M
    CGroup: /system.slice/docker.service
    		└─ 3456 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

If it does not show you a running service for the output then you'll have to troubleshoot further, but it may be as easy as starting the service.

Next you may also want to set it up so that Docker can be run without sudo or root access. You'll have to log out and back into the server after running this command for this to take effect.
```shell
sudo usermod -aG docker ${USER}
```

## Setting up Docker Compose

Create a directory for your Docker proxy and create a docker-compose.yml file within it.
```shell
mkdir ~/proxy
touch docker-compose.yml
```

I edit my files with Vim but you can you use whatever you want. We'll start by setting up our docker-compose file.
```yaml
# using version 2 of the docker-compose file layout
version: '2'

# defines which containers launch when running docker-compose
services:
  
  # initializes the Nginx reverse proxy container
  proxy:
    # image to launch
    image: jwilder/nginx-proxy
    # naming the container
    container_name: proxy
    # restarts the container unless it is stopped (manually or otherwise)
    restart: unless-stopped
    # creates a label so the Let's Encrypt container knows what proxy to connect to
    labels:
      com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy: "true"
    # connect the docker .sock, and 3 additional volumes that sync between proxy ~/proxy folder on server
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - certs:/etc/nginx/certs:rw
      - vhost.d:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - ./uploadsize.conf:/etc/nginx/conf.d/uploadsize.conf:ro
    # the proxy listens on ports 80 (http) and 443 (https)
    # all web traffic to the vps will have to come through the proxy so it knows how to route it correctly
    ports:
      - "80:80"
      - "443:443"
    # defining which networks the proxy container will communicate through
    # default network will handle traffic entering and exiting the VPS
    # proxy-tier network allows all contaienrs in the stack to communicate with each other
    networks:
      - "default"
      - "proxy-tier"

  # initializes the Let's Encrypt container
  proxy-letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: letsencrypt
    restart: unless-stopped
    environment:
      - NGINX_PROXY_CONTAINER=proxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    volumes_from:
      - "proxy"
    depends_on:
      - "proxy"
    networks:
      - "default"
      - "proxy-tier"
      
  # initializes portainer container, which lets you manage docker containers from a web interface
  portainer:
    image: portainer/portainer
    container_name: portainer
    restart: always
    environment:
      - VIRTUAL_HOST=portainer.DOMAIN.TLD
      - LETSENCRYPT_HOST=portainer.DOMAIN.TLD
      - LETSENCRYPT_EMAIL=EMAIL@ADDRESS.ME
    volumes:
      - ./portainer/:/data
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "9000:9000"

  # initializes nextcloud container
  nextcloud:
    image: nextcloud:apache
    container_name: nextcloud
    restart: unless-stopped
    environment:
      - VIRTUAL_HOST=nc.DOMAIN.TLD
      - LETSENCRYPT_HOST=nc.DOMAIN.TLD
      - LETSENCRYPT_EMAIL=EMAIL@ADDRESS.ME
      - PGID=999
      - PUID=1000
    volumes:
      - ./nextcloud:/var/www/html
    ports:
      - "666:80"
    networks:
      - proxy-tier
      - default
    depends_on:
      - "db"

  db:
    image: mariadb
    container_name: db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: SET_PASSWORD
      MYSQL_PASSWORD: SET_PASSWORD
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
    volumes:
      - ./db:/var/lib/mysql
    ports:
      - "3306:3306"

  # initializing a bitwarden_rs container
  bitwardenrs:
    image: bitwardenrs/server:latest
    container_name: bitwardenrs
    restart: always
    environment:
      - VIRTUAL_HOST=bw.DOMAIN.TLD
      - LETSENCRYPT_HOST=bw.DOMAIN.TLD
      - LETSENCRYPT_EMAIL=EMAIL@ADDRESS.ME
      - SIGNUPS_ALLOWED='true'
      - LOG_FILE='/data/bitwarden.log'
      - ADMIN_TOKEN=GENERATE_A_RANDOM_TOKEN_TO_PUT_HERE
    volumes:
      - ./bitwarden:/data
    ports:
      - "667:80"
    networks:
      - proxy-tier
      - default

# folders that get shared between the server's file system and the docker container for persistent storage
volumes:
  certs:
  vhost.d:
  html:

# internal networks
networks:
  proxy-tier:
```

Make sure to replace `DOMAIN.TLD` with your domain name in all sections of the config file. And replace `EMAIL@ADDRESS.ME` with your email address.
For the `db` container make sure to set database passwords. Replace the `SET_PASSWORDS` fields. 
To create a random token for the admin token you can use the following in the Ubuntu terminal:
```shell
< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c32
```

## Running Docker-Compose

Save the file and then you can execute the following to download the containers from the DockerHub and run them.
```shell
docker-compose up -d
```

## Accessing the Apps

When you go to those URLs now, `portainer.DOMAIN.TLD`, `nc.DOMAIN.TLD`, or `bw.DOMAIN.TLD`, you'll be able to reach the web interfaces of those applications. You should go into each of these interfaces and make sure you can set your password.
Also, keep in mind that for the bitwarden_rs app, people will be able to create accounts unless you disable that feature. You'll have to login to the admin panel for that at `bw.DOMAIN.TLD/admin` with that random admin token you generated before.

It may take a little while but make sure that the HTTPS is working for each of the apps and the certificate has been issued by Let's Encrypt. If that doesn't work you will have to do some more troubleshooting. Don't put off fixing that because otherwise traffic from your computer to the app's backend on the server is unencrypted and could be viewed by someone else with malicious intent.
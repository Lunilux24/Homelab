# Jellyfin Media Server

This will be my 3rd iteration of the Jellyfin Media Server. The original version was running on Windows bare metal off of my gaming PC. I used an Apache web server as a reverse proxy and whitelist. 

As soon as I bought my server hardware, I migrated to Ubunutu desktop, but had issues with the web UI leading me to factory reset the PC and try it all again, but this time, on Ubuntu Server. 

I will run Jellyfin in its own container and keep it separate from everything else.

## Setup

I started off by mounting the drives properly to my PC by following [this guide](https://www.wikihow.com/Linux-How-to-Mount-Drive) and then proceeded with writing the docker-compose.yaml file. It was important to configure the volumes for the config and cache correctly as well as use a bind mount to properly give the container access to my external hard drive that was holding all the movies.

```yaml
version: '3'
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfini1
    user: 1000:1000
    ports:
      - 8096:8096
    volumes:
      - ./config:/config 
      - ./cache:/cache
      - type: bind
        source: /media/jellyfind
        target: /media
    restart: 'unless-stopped'
```

## Errors Encountered

### ERROR: docker compose down, docker kill, docker stop, etc: permission denied

A Linux kernel security feature called AppArmor (Application Armor) enables the system administrator to limit the capabilities of individual programs by creating per-program profiles. This caused an interferance with docker and was solved by running the following command: ```sudo aa-remove-unknown```

[Source](https://medium.com/devops-technical-notes-and-manuals/how-to-solve-cannot-kill-docker-container-permission-denied-error-message-e3af7ccb7e29)

### ERROR: Docker container keyerror containerconfig

```docker-compose``` and ```docker compose``` are different and the former is now deprecated. To fix, I had to update to the new V2 version of docker using the following command.

```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

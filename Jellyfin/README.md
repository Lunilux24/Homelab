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

After setting up the docker compose file properly and troubleshooting the below bugs, I went through the initial setup guide on the web client and setup my libraries. 

![Jellyfin Dashboard Screenshot](../Photos/Screenshot%202025-08-02%20140405.jpg)

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

# Jellyfin Addons

Jellyfin has a variety of addons that can be installed to enhance the media server experience. Some popular addons include: Radarr, Sonarr, and Prowlarr for automated media management, which can be paired with download clients like qBittorrent in pair with a VPN called Gluetun for secure downloading. I built a separate stack for this which took a bit of time to get working properly due to the number of moving parts. 

I followed [this guide](https://trash-guides.info/) to set everything up. I will document the process I took here as well as the errors I encountered along the way.

## Setting up qBittorrent with Gluetun VPN

At first, I had issues getting qBittorrent to connect to the internet through the Gluetun VPN container. After some research, I found that I needed to set the network mode of the qBittorrent container to be the same as the Gluetun container. This allowed qBittorrent to use the VPN connection established by Gluetun. 

This is the first part of my compose file for the two containers:
```yaml
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    ports:
      - 8888:8888/tcp # HTTP proxy
      - 8388:8388/tcp # Shadowsocks
      - 8388:8388/udp # Shadowsocks
      - 8080:8080     # qBittorrent
      - 8191:8191     # flaresolverr
    volumes:
      - /opt/docker/gluetun:/gluetun
    environment:
      - VPN_SERVICE_PROVIDER=protonvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=your_private_key_here
      - SERVER_COUNTRIES=Canada
      - TZ=Etc/UTC

  qbittorrent:
    image: ghcr.io/linuxserver/qbittorrent
    container_name: qbittorrent
    network_mode: service:gluetun
    depends_on:
      - gluetun
    environment:
      - PUID=1000
      - PGID=1000
      - WEBUI_PORT=8080
    volumes:
      - ./qbittorrent:/config
      - /media/jellyfind/downloads:/downloads
    restart: unless-stopped
```

With this setup, qBittorrent was able to connect to the internet through the Gluetun VPN container successfully. Notice how the ```network_mode``` is set to ```service:gluetun``` in the qBittorrent container. This is what allows it to share the network stack with the Gluetun container. Also, we are exposing a port for qBittorrent's web UI (8080) in the Gluetun container so we can access it externally.

Depending on your VPN provider, you may need to adjust the environment variables in the Gluetun container to match your VPN credentials and preferences. In my case, I am using ProtonVPN with Wireguard protocol and connecting to servers in Canada. You can get your Wireguard private key from your ProtonVPN account dashboard.

After setting up qBittorrent with Gluetun, I was able to download files securely through the VPN connection. Next, I proceeded to set up Radarr and Sonarr to automate the downloading of movies and TV shows respectively. It allows you to manage your downloads and media library much more efficiently and when paired with Prowlarr, you can easily search for content across multiple indexers.

## Setting up Radarr, Sonarr, and Prowlarr

The next step was to set up Radarr, Sonarr, and Prowlarr to automate the downloading of movies and TV shows. These applications can be configured to monitor your media library and automatically download new content as it becomes available. 

⚠️ Note: Flaresolverr is a required component for some of these addons to work properly as it helps bypass Cloudflare protections on certain websites. As of right now, it is currently disabled by the developers with no plans on when it will be fixed. 

Here is the portion of my compose file responsible for these containers:

```yaml
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    network_mode: service:gluetun
    volumes:
      - ./flaresolver:/config
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - ./prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  sonarr:
    image: ghcr.io/linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - ./sonarr:/config
      - /media/jellyfind/Anime:/anime
      - /media/jellyfind/Shows:/tv
      - /media/jellyfind/downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped

  radarr:
    image: ghcr.io/linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - ./radarr:/config
      - /media/jellyfind/Movies:/movies
      - /media/jellyfind/downloads:/downloads
    ports:
      - 7878:7878
    restart: unless-stopped
```

You will notice that we expose ports for the web UIs of each application so we can access them externally. The volumes section is used to map the configuration files and media directories to the host system.

The last part of the setup is to configure each application through their respective web UIs. You will need to set up indexers, download clients, and media libraries in each application to get everything working properly. You can follow the guides on the [Trash Guides](https://trash-guides.info/) website for detailed instructions on how to do this.

This setup allows for a fully automated media server experience where new content is downloaded and organized without any manual intervention. It took some time to get everything configured properly, but the end result is worth it for the convenience it provides.
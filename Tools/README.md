# Network Tools

The tools in this stack will include:
    - Docker
    - Cockpit
    - Tailscale
    - Pi-hole
    - Portainer

As well as potentially:
    - Glance
    - Nginx Proxy Manager

## Setting up Docker

Docker was installed on my system when I initially setup the Ubuntu Server OS, but during my Jellyfin installation, I realized that I had installed a depricated version of docker compose. I updated to the current version via 

```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```


## Setting up Cockpit

Downloaded Cockpit via ```sudo apt install cockpit``` and started the service via ```sudo systemctl enable --now cockpit.socket```

No container needed for Cockpit.

## Setting up Tailscale
Originally, I had installed Tailscale via snap, but ran into issues where the Tailscale command (and other snap executible commands) couldn't be located in the shell's path. This is because I didn't have snap configured correctly, so instead of fixing it I decided it best to install Tailscale via their official container image.

```
# Remove snap version
sudo snap remove tailscale

# Install from official repo
curl -fsSL https://tailscale.com/install.sh | sh

# Enable and start
sudo tailscale up
```

After that it was as simple as logging in and and setting up my network.

## Setting up Portainer
I followed [this install guide](https://docs.portainer.io/start/install-ce) to setup Portainer. Configured the volume and then installed via the following command: 

```docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts```

Ran into no issues and reached via https://localhost:9443

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

## Setting up Pihole

I started off by navigating to the [official docker repo](https://github.com/pi-hole/docker-pi-hole) where I found a sample docker compose file there. After some edits, I ended up with this:

```
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "53:53/tcp"
      - "53:53/udp"
      # Default HTTP Port
      - "80:80/tcp"
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "443:443/tcp"
      # Uncomment the line below if you are using Pi-hole as your DHCP server
      #- "67:67/udp"
      # Uncomment the line below if you are using Pi-hole as your NTP server
      #- "123:123/udp"
    environment:
      # Set the appropriate timezone for your location (https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), e.g:
      TZ: 'America/Edmonton'
      # Set a password to access the web interface. Not setting one will result in a random password being assigned
      FTLCONF_webserver_api_password: 'REDACTED'
      # If using Docker's default `bridge` network setting the dns listening mode should be set to 'all'
      FTLCONF_dns_listeningMode: 'all'
    # Volumes store your data between container upgrades
    volumes:
      # For persisting Pi-hole's databases and common configuration file
      - './~/server/etc-pihole:/etc/pihole'
      # Uncomment the below if you have custom dnsmasq config files that you want to persist. Not needed for most starting fresh with Pi-hole v6. If you're upgrading from v5 you and have used this directory before, you should keep it enabled for the first v6 container start to allow for a complete migration. It can be removed afterwards. Needs environment variable FTLCONF_misc_etc_dnsmasq_d: 'true'
      #- './etc-dnsmasq.d:/etc/dnsmasq.d'
    # cap_add:
      # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
      # Required if you are using Pi-hole as your DHCP server, else not needed
      # - NET_ADMIN
      # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
      # - SYS_TIME
      # Optional, if Pi-hole should get some more processing time
      # - SYS_NICE
    restart: unless-stopped
```

The entire file has a lot of bloat that I didn't end up actually needing. The most important part was ensuring that the DNS ports were configured properly for both TCP and UDP, HTTP port was = 80 so we can access the webclient and that the ```FTLCONF_webserver_api_password: " is set correct. It is also important to mention that the volume for the config file needs to be mounted correctly as the container will need proper access to that file. 

When I ran the container I got a message saying "failed to bind host port". Apparently Linux has a DNS stub listener enabled by default that uses port 53 so I had to disable that next. The first step was to disable the DNS Stub Listener by changing its' value in the config file. 

```
# Disable DNS Stub Listener
sudo nano /etc/systemd/resolved.conf
```

You are going to want to look for a line that may be commented out. It will say ```#DNSStubListener=Yes```. Just uncomment that and change it to "No". Moving on, you are going to want to remove a different config file and make a link to it as follows.

```
# Remove existing resolv config
sudo rm /etc/resolv.conf

# Make the symbolic link to a different location
sudo ln -s /run/systemd/resolve/resolv.conf  /etc/resolv.conf

# Restart the services
sudo systemctl restart systemd-resolved
sudo systemctl daemon-reload
```

Pihole should now run properly!

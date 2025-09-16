# Homelab
This is the github repository for my Homelab. At the moment, I am running Ubuntu Server + Cockpit for remote monitoring and a frontend interface. I plan on using this repository to store important files such as docker-compose files, keep track of important links, plan future projects and document my progress and the things I learn along the way.

## My Current Setup

- LENOVO 10RRS0B500 ThinkCentre M920q
- CPU: 6x Intel(R) Core(TM) i5-8500T CPU @ 2.10GHz
- Memory: 2x 16gb DDR4
- OS: Ubuntu 24.04.2 LTS
- Drive 1: WD 1TB External HDD
- Drive 2: WD 3TB External HDD

## Applications

### Current Applications
  - Cockpit (Interface)
  - Jellyfin (Media Server)
  - Potainer.io (Container Manager)
  - Pi-hole (Network Monitor)
  - Nextcloud (File Server)
  - Immich (Image File Server)
  - Glance (Web Interface)
  - Karakeep (Bookmarker)

### How It All Works
I am currently running all the above applications in Docker containers on my server. To remotely access them, I have a personal Tailnet setup through Tailscale. The reason I went with this route was because unlike the first iteration of my server, I added a lot more programs and felt like this would not only be the safer alternative, but the easier alternative as well. Using Tailscale, I just turn on my VPN on my phone and my IP address is masked as if I am on my home network! This makes connecting to my applications incredibly easy as all I have to do now is use the Tailnet IPv4 address of my server for quick and easy access. 

## What's Next?
Now that I have setup all the applications that I had planned for this home server, the next step will be bringing enhancements to make the experience even better. That means potentially adding something like Kubernetes for better upkeep, playing around with the addresses of my application so that I don't have to remember the port numbers and adding new applications like N8N for automation.

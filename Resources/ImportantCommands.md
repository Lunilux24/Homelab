Linux Commands

```bash
# Enter root terminal
sudo -i

# Change name of file
mv oldname.txt newname.txt

# Reboot PC
sudo reboot
```

Nginx Commands

```bash
# Check nginx running status
sudo systemctl status nginx

# nginx file path
/etc/nginx

# Restart nginx
service nginx restart
```

Docker Commands

```bash
# Show all running docker commands
docker ps -a

# Run the docker container for Jellyfin
cd Documents/jellyfin | docker compose up --build -d

# Enter docker [program] cli (ex. FileBrowser)
docker exec -it filebrowser
```


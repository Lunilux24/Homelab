# File Server

The applications in this stack will include all file management or productivity related applications. The list includes:
    - File Browser
    - Immich
    - Docmost
    - Hoarder

This stack is intended to move a lot of my productivity and file storage apps to open source and free applications. Nextcloud will replace my Google Drive, Immich will replace my iCloud Photos, Docmost to replace my OneNote and Hoarder as a storage for all the important articles and guides I will inevitably come across during my Homelab journey. 

## Setting up File Browser

Setting up File Browser involved crafting up a docker compose file and running it. At the moment, I am having an issue where restarting the container deletes all of the users and switches the admin password as well. I haven't had the chance to look into this yet. 

Here is the compose file that I used:

```
version: '3.3'

services:
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    user: "1000:1000"
    ports:
      - 8080:80
    volumes:
      - /media/filebrowser_files:/srv
      - /media/jellyfin_files:/srv/jellyfin_files
      - /~/server/filebrowser/data/database.db:/database.db
      - /~/server/filebrowser/config/filebrowser.json:/.filebrowser.json
    restart: always
```

After a simple crash the first time where my database.db path was inaccessible, the interface was accessible via localhost:8080 and I proceeded to login with "admin" as my username and the randomly generated password that created on initial build. You can find it by running ```docker logs```. Just ensure that your container is up and running otherwise, you won't be able to see the output.

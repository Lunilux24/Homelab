# Productivity Server

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

## Errors Encountered

The most difficult part (or so I thought) was mounting the volumes correctly and ensuring that the filebrowser.json and database.db folders were in the right locations. Once I got it running however, I came to realize that I wasn't able to upload, edit, or create folders or files. Instantly, I thought to check the logs and quickly found that I had the wrong permissions. This eventually led to me checking and realizing that my docker compose file was running the container with UID/GID = 1000 and the folder that it was trying to write to was owned by the root user and wouldn't allow write access. As a result I began looking for a way to change the permissions.

The first option was to run the container in sudo (UID/GID = 0), but I was worried about potential invaders getting access to too much power; especially with an application like a file browser. This led to the second option. Unmount the drive and change the permissions using chmod. Ultimately this led no where because I quickly came to realize that the format of my drive was exFAT. The reason that this was a problem was because exFAT does not allow traditional linux commands. 



# Productivity Stack

The applications in this stack will include all file management or productivity related applications. The list includes:
    - Immich
    - Karakeep
    - Nextcloud
    - Appflowy
    
This stack is intended to move a lot of my productivity and file storage apps to open source and free applications. Nextcloud will replace my Google Drive, Immich will replace my iCloud Photos, Docmost to replace my OneNote and Hoarder as a storage for all the important articles and guides I will inevitably come across during my Homelab journey. 


## Setting up Immich

Immich was a lot easier than I had anticipated to setup. It consisted of following [this quickstart guide](https://immich.app/docs/overview/quick-start/) and tweaking a few environment variables. After that, all the configuration was done from within the app! Immich is without a doubt the most useful application in my homelab and it gets the certified "can't live without" seal of approval that only Jellyfin had up until this point.

## Setting up Karakeep

Followed the install guide [here](https://docs.karakeep.app/installation/docker/)

## Setting up Nextcloud

To setup Nextcloud, I decided to write my own docker compose file. Before that however, I had to create directories for persistent data store. 

```
sudo mkdir -p /media/nereusd/nextcloud/{db,html}
sudo chown -R 1000:1000 /media/nereusd/nextcloud
```

Next, I wrote my docker compose file. I wasn't really sure if this was the best way to do it, however due to Nextcloud having a number of services that were unnecessary to me, I decided writing my own bare bones compose file was the better option. 

```
services:
  db:
    image: mariadb:11
    container_name: nextcloud-db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - /media/nereusd/nextcloud/db:/var/lib/mysql    # Volume path for DB
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=password

  app:
    image: nextcloud:29
    container_name: nextcloud-app
    restart: unless-stopped
    ports:
      - "8080:80"    # Changed the ports as these were being used by Pi-hole
    links:
      - db
    volumes:
      - /media/nereusd/nextcloud/html:/var/www/html    # Volume path for the base files
      - /media/jellyfind:/jellyfin_files               # Volume path for Jellyfin
      - /media/nereusd/files:/data-secondary           # Volume path for my files
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=password
```

This container actually ran first try which was kind of surprising to me, but I couldn't do anything on the app due to this error: "Please change the permissions to 0770 so that the directory cannot be listed by other users." After messing around with group perms on both my server and docker, I found out you can disable it by instead accessing the config.php file (which in this case would be in the ```/media/nereusd/nextcloud/html/.../config.php``` file path and adding in ```'check_data_directory_permissions' => false,```. Shoutout u/d_popov for that one.

I still had mounting problems to fix though and it turned out that on Nextcloud, you have to enable something called External Storage. It's a plugin that you can enable by going to the apps tab on the web client. Looks like there is a lot of potential to add some cool features to Nextcloud so I will look into adding more of these apps later on. 

## Setting up Appflowy

This one is still a work in progress. I have the container running, but the configuration settings are totally wrong. I will update this once it is fixed and I know what the issue is. 

## Setting up File Browser (Abandoned)

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

In order to fix this, I had to change the format of the drive from ExFat to NTFS. The procedure was as follows:

```
# Make a backup of the current drive
sudo mkdir -p /media/nereusd/jellyfin_backup
sudo cp -a /media/jellyfind/. /media/nereusd/jellyfin_backup/

# Unmount the drive
sudo umount /media/jellyfind

# Format the drive
sudo mkfs.ntfs -f /dev/sdb2

# Remount the drive
sudo mount -t ntfs /dev/sdb2 /media/jellyfind

# Restore backup
sudo cp -a /media/nereusd/jellyfin_backup/. /media/jellyfind/
```

In order to remount the drive, I needed to reboot the server due to the drive being "busy". I wasn't able to pinpoint what was using the drive so a quick reboot solved the problem.


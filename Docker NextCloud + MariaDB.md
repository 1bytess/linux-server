# Nextcloud and MariaDB for Docker Container Installation

## Table of Content

- [Instalation]
- [Configuration]
- [Initial Setup]

## Installation

Using Docker Compose or Portainer Stack

1. Create Directory path for both **Nextcloud** and **MariaDB**

```bash
sudo mkdir -p /srv/appdata/{nextcloud,mariadb}
```

1. Set folder owner:

```
sudo chown -R <user>:<user> /srv/appdata/nextcloud/
sudo chmod -R 750 /srv/appdata/nextcloud
```

1. Create Docker Network so eveything run under the same network

```
sudo docker network create nextcloud-network
```

1.  Create `Docker Compose` or Create new `Portainer Stack`: 

- **Nextcloud**

```b
version: '3.9'
services:
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1000 #Change this using `sudo id -u <username>`
      - PGID=1000 #Change this using `sudo id -g <username>`
      - TZ=Asia/Seoul
      - MYSQL_HOST=mariadb
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=<nextcloud-sql-passwd> #Change This!
    volumes:
      - /srv/appdata/nextcloud/app:/var/www/html
      - /srv/appdata/nextcloud/config:/config
      - /srv/appdata/nextcloud/data:/data
    ports:
      - 7070:443 #Change 7070 to any port available
    networks:
      - nextcloud-network
    restart: unless-stopped

networks:
  nextcloud-network: #Connect to Network we created
    external: true
```

- **MariaDB**

```b
version: '3.9'
services:
  mariadb:
    image: lscr.io/linuxserver/mariadb:latest
    container_name: mariadb
    environment:
      - PUID=1000 #Change this using `sudo id -u <username>`
      - PGID=1000 #Change this using `sudo id -g <username>`
      - TZ=Asia/Seoul
      - MYSQL_ROOT_PASSWORD=<super_secure_root_password> #Change this
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=<nextcloud-sql-passwd> #Same as
    volumes:
      - /srv/appdata/mariadb/config:/config
    ports:
      - 3306:3306
    networks:
      - nextcloud-network
    restart: unless-stopped

networks:
  nextcloud-network: #Connect to Network we created
    external: true
```

- Redis (for memcache.locking)

```b
version: '3.9'

services:
  redis:
    image: redis:latest
    container_name: redis
    networks:
      - nextcloud-network
    ports:
      - "6379:6379"
    volumes:
      - /srv/appdata/redis:/data

volumes:
  redis_data:

networks:
  nextcloud-network:
    external: true
```

## Configuration

To run `occ` on docker should use:

```
sudo docker exec -it nextcloud occ <db: add-missing-indices>
```

  
To edit `php`, is one directory

```bash
nano /srv/appdata/nextcloud/config/www/nextcloud/config/config.php
```

Add this to new line:

```
  'default_phone_region' => 'KR',
  'maintenance' => false,
  'maintenance_window_start' => 1,
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => array(
     'host' => '<redis-network-ip-address>',
     'timeout' => 0.0,
     'password' => '',
      ),
```

  
Edit `Strict-Transport-Security` on docker is at:

```bash
sudo nano /srv/appdata/nextcloud/config/nginx/ssl.conf
```

and un-comment

```
add_header Strict-Transport-Security "max-age=63072000" always;
```

lastly, restart container

```bash
docker restart nextcloud
```

## Initial Setup

![NextCloud Admin Account Creation](.attachments.847/Create-Nextcloud-admin-account-1024x723.jpg.jpg)

`username`, `Password` will be used as account login and admin  
  
`Data Folder`: Use the folder under nextcloud account under path `/path`  
  
`Database password`: same as in the docker stack set in **Nextcloud** compose file.  
  
`Database host`: use *localhost*, or if not working use the ip address in the nextcloud-network using:

```bash
docker network inspect nextcloud-network
```

will show output:

```
"<CONTAINER-ID>": {
                "Name": "mariadb",
                "EndpointID": "<Endpoint-ID>",
                "MacAddress": "<mac-address>",
                "IPv4Address": "172.21.0.4/16", # Copy This 
                "IPv6Address": ""
            },
```

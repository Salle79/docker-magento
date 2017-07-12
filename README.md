# Docker image for Magento 1.x

This repo is loosly based  onDocker image for [Magento 1.x](http://magento.com/).

#### Please note

> The primary goal of this repo is to create Docker images for Magento 1.x development and testing, especially for extensions and themes development. It lacks essential support for production deployment, e.g. Varnish and Redis. Use this wisely for production deployment.

> This repo is only for Magento 1.x. If you are looking for Magento 2.x, check out [alexcheng1982/docker-magento2](https://github.com/alexcheng1982/docker-magento2).

## Magento versions

Version | Git branch | Tag name
--------| ---------- |---------
1.9.2.4 | master     | latest
1.9.1.1 | 1.9.1.0    | 1.9.1.0
1.8.1.0 | 1.8.1.0    | 1.8.1.0
1.7.0.2 | 1.7.0.2    | 1.7.0.2
1.6.2.0 | 1.6.2.0    | 1.6.2.0

## How to use

### Use as standalone container

You can use `docker run` to run this image directly.

```bash
docker run -p 80:80 alexcheng/magento
```

Then finish Magento installation using web UI. You need to have an existing MySQL server.

Magento is installed into `/var/www/htdocs` folder.

### Use Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is the recommended way to run this image with MySQL database.

A sample `docker-compose.yml` can be found in this repo, it includes a complete stack: Magento, Mysql and Redis.

```yaml
wversion: '2'
 volumes:
   data-magento: {}
   data-log: {}
 services:
   web:
     container_name: web-node
     build: .
     ports:
       - "80:80"
     networks:
           vpcbr:
             ipv4_address: 10.5.0.5
     volumes:
       - ~/workspace/docker-magento/www:/var/www/htdocs
     extra_hosts:
      - "local.magento:172.19.0.4"
     links:
       - mysql
       - redis
     env_file:
       - env
     depends_on:
       - mysql
       - redis
   mysql:
     container_name: mysql-node
     image: mysql
     networks:
               vpcbr:
                 ipv4_address: 10.5.0.6
     env_file:
           - env
     volumes:
       - data-magento:/var/lib/mysql
       - data-log:/var/log
       - ./data/database.sql:/docker-entrypoint-initdb.d/init.sql
   redis:
     container_name: redis-node
     image: redis:latest
     networks:
               vpcbr:
                 ipv4_address: 10.5.0.8
 
 networks:
   vpcbr:
     driver: bridge
     ipam:
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

To start it up, use `docker-compose up -d`.

## Modman
Modman is a [Magento module manager](https://github.com/colinmollenhour/modman) that allows you to leave your work siloed from the actual Magento codebase via symlinks. With modman, you can sync plugin or theme work without keeping a persistent volume (or using a hidden volume).

```bash
# from htdocs
modman init
modman link /path/to/plugin
```
And to update symlinks:
```bash
modman deploy
```

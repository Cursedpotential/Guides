# Wordpress on Docker

### This file will setup Wordpress, MySQL & PHPMyAdmin with a single command.

This files also uses 2 networks. I did this so that nginx or another reverse proxy could be added to the stack and then the WP container could be taken off the outward facing "web" network. I might add another more simple config later. 

#### Steps:

Create a docker network called "web":
```
$ docker network web
```
Create a new folder to house your docker-compose file and mount our WP volumes in. I will be using $HOME/wordpress_docker/:

```
 $ cd && mkdir wordpress_docker 
 $ cd wordpress_docker	
```

Add the code below to a file called "docker-compose.yaml" and run the command:
```
$ docker-compose up -d
```
After that you can go to http://localhost:8080 to finish setting up wordpress. PHPMyAdmin is available at http://localhost:8081

```
version: '3'

services:
  wp:
    image: wordpress:latest # https://hub.docker.com/_/wordpress/
    ports:
      - 8080:80 # change ip if required
    restart: always
    volumes:
      - ./config/php.conf.ini:/usr/local/etc/php/conf.d/conf.ini
      - ./wp-app:/var/www/html # Full wordpress project
      #- ./plugin-name/trunk/:/var/www/html/wp-content/plugins/plugin-name # Plugin development
      #- ./theme-name/trunk/:/var/www/html/wp-content/themes/theme-name # Theme development
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: P@ssw0rd123
    depends_on:
      - db
    networks:
      - web            
      - internal
 
  db:
    image: mysql:latest # https://hub.docker.com/_/mysql/ - or mariadb https://hub.docker.co>
    volumes:
      - ./wp-data:/docker-entrypoint-initdb.d
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssw0rd123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: P@ssw0rd123
    networks:
      - internal       

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: pma
    links:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    restart: always
    ports:
      - 8081:80
    networks:
      - web
      - internal
volumes:
  db_data:

networks:
  internal:
     internal: true
     attachable: true # compose version 1.29
  web:
    external: true

```
> Written with [StackEdit](https://stackedit.io/).
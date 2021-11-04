# Simple Wordpress Install on Docker

### This file will setup Wordpress & MySQL with a single command.

#### Steps:

Create a new folder to house your docker-compose file and mount our WP volumes in. I will be using $HOME/wordpress_docker/:
```
$ cd && mkdir wordpress_docker 
$ cd wordpress_docker	
```
Add the code below to a file called "docker-compose.yaml" and run the command:
```
 docker-compose up -d 
```

After that you can go to http://localhost:8080 to finish setting up wordpress. 

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
      - web     

volumes:
  db_data:

networks:
  web:
     attachable: true
```
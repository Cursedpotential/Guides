# Install Ghost CMS on Docker
##### Includes config for second reverse proxy through NGINX


## Install Ghost

The Ghost installation will comprise of three components or containers - Ghost package, a database server (MySQL), and a web server (Nginx). All these services will be installed using a single Docker compose file.  I have included a second cerbot and nginx config to act as a reverse proxy to a wordpress instance I have installed on my network. If you don't need a second site then ignore the commented sections referring to this.

### Install Certbot and obtain the SSL certificate

Before we proceed, we need to install the Certbot tool and install an SSL certificate for our domain.

To install Certbot, we will use the Snapd package installer. Certbot's official repository has been deprecated and Ubuntu's Certbot package is ooold. Snapd always carries the latest stable version of Certbot and you should use that.

Ensure that your version of Snapd is up to date.

```shell
$ sudo snap install core 
$ sudo snap refresh core

```

Remove any old versions of Certbot.

```shell
$ sudo apt remove certbot

```

Install Certbot.

```shell
$ sudo snap install --classic certbot

```

Use the following command to ensure that the Certbot command can be run by creating a symbolic link to the  `/usr/bin`  directory.

```shell
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot

```

Generate an SSL certificate.

```shell
$ sudo certbot certonly --standalone -d ghost.msalemtech.com
```
>If using the second app below as I did with wordpress, here is where you would create a second cert for your second domain.

The above command will download a certificate to the  `/etc/letsencrypt/live/ghost.msalemtech.com`  directory on your server.

### Create Docker Compose File

1. Create a directory to store and launch your Docker compose file from.

```shell
$ mkdir ghost && cd ghost

```

2. Create a file named  `docker-compose.yml`  and open it . (I'm using nano for this)

```shell
$ nano docker-compose.yml
```
3. Paste the following code in the file. Change any user names and passwords for security. Also change any instances of a domain name to match your config.

```
version: '3.1'
services:

  ghost:
    image: ghost:latest
    restart: always
    depends_on:
      - db
    #ports:
    #  - 8080:2368
    environment:
      # see https://ghost.org/docs/config/#configuration-options
      database__client: mysql
      database__connection__host: db
      database__connection__user: root
      database__connection__password: P@ssw0rd123
      database__connection__database: ghost
      #mail__transport: SMTP
      #mail__options__host: {Your Mail Service host}
      #mail__options__port: {Your Mail Service port}
      #mail__options__secureConnection: {true/false}
      #mail__options__service: {Your Mail Service}
      #mail__options__auth__user: {Your User Name}
      #mail__options__auth__pass: {Your Password}
      # this url value is just an example, and is likely wrong for your environment!
      url: https://localhost
      # contrary to the default mentioned in the linked documentation, this image defaults to NODE_ENV=production (so development mode needs to be explicitly specified if desired)
      #NODE_ENV: development
    volumes:
       - ./content:/var/lib/ghost/content
  db:
     image: mysql:5.7
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: P@ssw0rd123
     volumes:
       - ./mysql:/var/lib/mysql
  nginx:
     build:
       context: ./nginx
       dockerfile: Dockerfile
     restart: always
     depends_on:
       - ghost
     ports:
       - "80:80"
       - "443:443"
     volumes:
       - /etc/letsencrypt/:/etc/letsencrypt/
       - /usr/share/nginx/html:/usr/share/nginx/html
```
>The mail server info is commented out. If you would like to set it up now, uncomment the lines and fill it in with your server info. See the referenced pages below for more help with email.

4. Create directories for all the bind mounts described above (except for  `/etc/letsencrypt`, which was already created when we created the certificate before)

```shell
$ cd ~/ghost
$ mkdir content
$ mkdir mysql
$ sudo mkdir -p /usr/share/nginx/html
```
### Create the Nginx Docker Image

The Docker compose file that we created relies on the Nginx Docker image. To make it work, we need to include a customized configuration file for Nginx which will work with Ghost.

1. Create a directory for this image in the current directory.

```shell
$ mkdir nginx

```

2. Create a file named  `Dockerfile`  in this directory.

```shell
$ touch nginx/Dockerfile

```

3. Paste the following code in the  `Dockerfile`.

```dockerfile
FROM nginx:latest
RUN rm /etc/nginx/conf.d/default.conf
COPY ghost.conf /etc/nginx/conf.d
#COPY wordpress.conf /etc/nginx/conf.d
```

The above code instructs Docker to use the latest Nginx image. It also deletes the default Nginx configuration file and copies the custom configuration file that we have created for our Ghost CMS. 

>I have included a comment for adding a second file. This one for wordpress, as I run a wordpress install also. I will include an nginx reverse proxy config file below if you would like to adapt it for a  second app you have running on your network. Otherwise you can ignore it or leave it commented out for later.

4. Create a file named  `ghost.conf`  in the  `nginx`  directory.

```shell
$ nano nginx/ghost.conf

```

5. Paste the following code in the  `ghost.conf`  file. Replace all domain entries  with your domain.
```
server {
  listen 80;
  listen [::]:80;
  server_name ghostmsalemtech.com;
  # Useful for Let's Encrypt
  location /.well-known/acme-challenge/ { root /usr/share/nginx/html; allow all; }
  location / { return 301 https://$server_name$request_uri; }
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name ghost.msalemtech.com;
    
  access_log /var/log/nginx/ghost.access.log;
  error_log /var/log/nginx/ghost.error.log;
  client_max_body_size 20m;

  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_session_timeout 1d;
  ssl_session_cache shared:SSL:10m;

  ssl_certificate     /etc/letsencrypt/live/ghost.msalemtech.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/ghost.msalemtech.com/privkey.pem;

  location / {
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://ghost:2368;
  }
}
```
##### Optional Config
If you are using a second app on your network that you would like to utilize this nginx installation as a proxy for this is where you would configure it. Below is my wordpress nginx config file that was added to my Dockerfile. To use it create a file called wordpress.conf in the nginx folder and paste the config below. Dont forget to change the domains to match yours.
```
$ nano nginx/wordpress.conf
```
```
server {
  listen 80;
  listen [::]:80;
  server_name wordpress.msalemtech.com;
  # Useful for Let's Encrypt
  location /.well-known/acme-challenge/ { root /usr/share/nginx/html; allow all; }
  location / { return 301 https://$server_name$request_uri; }
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name wordpress.msalemtech.com;
    
  access_log /var/log/nginx/wordpress.access.log;
  error_log /var/log/nginx/wordpress.error.log;
  client_max_body_size 20m;

  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_session_timeout 1d;
  ssl_session_cache shared:SSL:10m;

  ssl_certificate     /etc/letsencrypt/live/wordpress.msalemtech.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/wordpress.msalemtech.com/privkey.pem;

  location / {
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://192.168.1.25:8080;
  }
}
```
### Start your stack!
```
$ docker-compose.yml up -d
```
```
$ docker -ps
```

If your blog doesn't  start enter the `docker-compose up`  command without the -d to see any errors.

### Start Blogging
In this example my blog in now available at https://ghost.msalemtech.com 

I will work on a more generalized install as well as a localhost install.


References: https://www.howtoforge.com/how-to-install-ghost-cms-with-docker-on-ubuntu-20-04/Empty File
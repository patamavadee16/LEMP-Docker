# LEMP-Docker

### Config Docker-compose เพื่อให้งาน Nginx Web Server

สร้าง Project ใหม่ภายใน Folder nginx_dock ประกอบด้วย

* docker-compose.yml
* static-html
    * index.html

ภายในไฟล์ docker-compose.yml
```Makeup
version: '3'

services:
  nginx:
    container_name: nginx
    restart: unless-stopped
    image: nginx:stable-alpine
    volumes:
      - ./static-html:/usr/share/nginx/html
    ports:
      - "80:80"

networks:
  default:
    external:
      name:
        web_network
```
ภายในไฟล์ index.html
```html
Hello Nginx
```

แล้วรันContainer ด้วย docker-compose ด้วยคำสั่ง
```
docker-compose up -d
```

### Config Nginx และ php FPM Container

สร้าง Project ใหม่ภายใน Folder lemp_dock ประกอบด้วย

* docker-compose.yml
* html
    * index.php
* nginx
    * conf
        * nginx.conf
    * conf.d
        * default.conf

ภายในไฟล์ docker-compose.yml (ของFolder lemp_dock)
```Makeup
version: '3'

services:
  php:
    container_name: lemp_php
    image: php:7.4-fpm-alpine
    restart: unless-stopped
    volumes:
      - ./html/:/var/www/html
    expose:
      - "9000"

  nginx:
    container_name: lemp_nginx
    image: nginx:stable-alpine
    restart: unless-stopped
    volumes:
      - ./html/:/var/www/html

      - ./nginx/conf/nginx.conf:/etc/nginx/conf/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro

    ports:
      - "80:80"

networks:
  default:
    external:
      name:
        web_network
```
ภายในไฟล์ index.php
```
<?php phpinfo();?>
```

ภายในไฟล์ nginx.conf
```
worker_processes 1;
daemon off;
events {
    worker_connections 1024;
}
error_log   /var/log/nginx/error.log warn;
pid         /var/run/nginx.pid;
http {
    include /etc/nginx/conf/mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request"'
    '$status $body_bytes_sent "$http_referer"'
    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    sendfile on;
    #tcp_nopush on;
    keepalive_timeout 65;
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    # tells the server to use on-the-fly gzip compression.
    include /etc/nginx/conf.d/*.conf;
}
```

ภายในไฟล์ default.conf
```
server {
   charset utf-8;
   client_max_body_size 128M;
   listen 80; ## listen for ipv4
   #listen [::]:80 default_server ipv6only=on; ## listen for ipv6
   
   root /var/www/html;
   index index.php;

   location / {
       # Redirect everything that isn't a real file to index.php
       try_files $uri $uri/ /index.php$is_args$args;
   }

   # uncomment to avoid processing of calls to non-existing static files by Yii

   #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
   #    try_files $uri =404;
   #}

   #error_page 404 /404.html;

   location ~ \.php$ {
       include fastcgi_params;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_pass php:9000;
       try_files $uri =404;
   }

   location ~ /\.(ht|svn|git) {
       deny all;
   }
}
```

แล้วรันContainer ด้วย docker-compose ด้วยคำสั่ง
```
docker-compose up -d
```
## Config MariaDB Container
* docker-compose.yml
* html
    * index.php
* nginx
    * conf
        * nginx.conf
    * conf.d
        * default.conf
* *mariadb*
    * *data*
    * *initdb*
        * *tinanic.sql*
    * *backup*
* *php*
    * *Dockerfile*

จากนั้นแก้ไขไฟล์docker-compose.yml (ของFolder lemp_dock) ให้เป็นดังนี้
```
version: '3'

services:
  php:
    container_name: lemp_php
    build: php/
    restart: unless-stopped
    volumes:
      - ./html/:/var/www/html
    expose:
      - "9000"
    depends_on:
      - db

  nginx:
    container_name: lemp_nginx
    image: nginx:stable-alpine
    restart: unless-stopped
    volumes:
      - ./html/:/var/www/html

      - ./nginx/conf/nginx.conf:/etc/nginx/conf/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro

    ports:
      - "80:80"
      
  db:
    container_name: lemp_mariadb
    image: mariadb:latest
    restart: unless-stopped
    volumes:
      - ./mariadb/initdb/:/docker-entrypoint-initdb.d
      - ./mariadb/data/:/var/lib/mysql/
    environment:
      - MYSQL_ROOT_PASSWORD=devops101
      - MYSQL_DATABASE=devops_db
      - MYSQL_USER=devops
      - MYSQL_PASSWORD=devops101

networks:
  default:
    external:
      name:
        web_network
```
ภายใน Dockerfile จะมีการติดตั้ง mysqli เพื่อเรียก Mariadb จาก php
```
FROM php:7.4-fpm-alpine

RUN docker-php-ext-install mysqli
```
แล้วแก้ไขไฟล์ index.php
```
<?php
   $servername = "db";
   $username = "devops";
   $password = "devops101";

   $dbhandle = mysqli_connect($servername, $username, $password);
   $selected = mysqli_select_db($dbhandle, "titanic");
   
   echo "Connected database server<br>";
   echo "Selected database";
?>
```

สร้าง php Image ด้วยคำสั่ง
```
docker-compose build
```
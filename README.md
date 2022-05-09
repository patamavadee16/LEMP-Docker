# LEMP-Docker

## Config Docker-compose เพื่อให้งาน Nginx Web Server

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


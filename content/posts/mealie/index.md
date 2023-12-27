---
title: "Mealie Installation Steps"
date: 2023-12-27T19:58:17+03:00
authors: ["Gokberk Gunes", ]
tags: [server]
#header_imageX: ./images/title.webp
draft: true
---

1. Create a folder to hold docker-realted files.
2. In this folder, create a file as: `docker-compose.yaml`.
3. Change the content of this file using the https://nightly.mealie.io/documentation/getting-started/installation/sqlite .
4. The file's variables could be modified. These variables are given in: https://nightly.mealie.io/documentation/getting-started/installation/backend-config/ .
5. In this tutorial
```yaml
---
version: "3.7"
services:
  mealie:
    image: ghcr.io/mealie-recipes/mealie:v1.0.0-RC1.1
    container_name: mealie
    ports:
        - "9925:9000" #
    deploy:
      resources:
        limits:
          memory: 1000M #
    volumes:
      - mealie-data:/app/data/
    environment:
    # Set Backend ENV Variables Here
      - ALLOW_SIGNUP=false
      - APP_DEFAULT_DARK_MODE=true
      - PUID=1000
      - PGID=1000
      - MAX_WORKERS=1
      - WEB_CONCURRENCY=1
      - BASE_URL=https://mealie.berksen.com
    restart: always

volumes:
  mealie-data:
    driver: local
```
4. Run the docker installation command: `doas docker-compose up -d`. Also, you
   can use this command to restart after updating above settings.
5. After the installation is done, you may reach the server using
   `http://192.168.0.2:9925`. Use `changeme@example.com` and `MyPassword` as
   the admin account login credentials. Since these credentials belongs to admin account, they must be changed.
   These variables may change; refer to documentation[^1].
6. Change the default credentials for admin account in the website under
   settings tab.
7. Get a `CNAME` in your domain. It is `mealie.berksen.net` in this case.
8. Add below to `http` part of `nginx.conf`.
```
server {
	server_name mealie.berksen.net;
	location / {
		proxy_pass http://127.0.0.1:9925;
	}
}
```

[^1]: [Mealie Documentation](https://nightly.mealie.io/documentation/getting-started/installation/installation-checklist/].)

---
title: "Mealie Installation Steps"
date: 2023-12-27T19:58:17+03:00
authors: ["Gokberk Gunes", ]
tags: [server]
#header_imageX: ./images/title.webp
draft: false
---

## Installation of Mealie
In this guide, we will install mealie with docker given that how
straightforward it is. The first step is to create  a folder to hold all the
setting file that is used to run docker. Accordingly, create a folder and
a file under this folder using commands:
```sh
mkdir mealie
touch mealie/mealie-settings.yml
```

We should fill this file with our desired settings. At the time of writing of
this guide , I used below to create my mealie instance:
```yaml
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
On mealie's website, the suggested settings can be found to fill docker settings file, `mealie-settings.yml`. The direct link to SQLite based suggested settings are given in [here](https://nightly.mealie.io/documentation/getting-started/installation/sqlite).
Moreover, information on the variables be found in this
[link](https://nightly.mealie.io/documentation/getting-started/installation/backend-config).


The next step is to deploy the mealie using docker. To start the installation,
run the docker installation command: `doas docker-compose up -d`. Additionally,
we may use this command to restart after changing the settings in
`docker-compose.yaml`.

After the installation is done, you may reach the server using
`http://local_ip_of_server:9925`, e.g., `http://192.168.0.10:9925`. If we are
installing mealie to our own computer, we can reach to mealie by going to
`http://localhost:9925`. We will use the admin account to log in to change
settings. As the login credentials, we should use `changeme@example.com` and
`MyPassword`. Since these credentials belong to admin account, we must be
change them. Note that, if the credentials are not working, we can refer to
[mealie
documentation](https://nightly.mealie.io/documentation/getting-started/installation/installation-checklist/].).

After logging in, we should go to settings tab and change the credentials for
admin account there. Additionally, since we have disabled registration with
`ALLOW_SIGNUP=false` setting earlier, we need to create users[^1].

## Host Mealie on the World Wide Web
First, we must get a `CNAME` in our domain. For this, we should go to the
domain provider and register a `CNAME` as we wish. In this tutorial, this is
`mealie.berksen.net`.

Now, we should tell `nginx` to redirect all the visitors of
`mealie.berksen.net` to our local website. This can be set by appending the
piece of setting given below to `nginx.conf`.
```nginx
http {
	server {
		server_name mealie.berksen.net;
		location / {
			proxy_pass http://127.0.0.1:9925;
		}
	}
}
```

### Enable Encryption (https)
Encryption is a vital requirement for any activity online. For `mealie`, this is
done very easily by using the great tool `certbot.` Simply, we should run `certbot` as
the root user to generate a certificate. To do this, run `doas certbot` or
`sudo certbot` or `su -c certbot`, and generate the `https` certificate.


[^1]: This is done to avoid setting mail server and settings related to it.

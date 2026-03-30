---
title: "Docker Notes"
date: 2026-03-30T08:50:25-05:00
authors: ["Gokberk Gunes", ]
tags: []
header_image:
draft: false
---

## Installing Docker
Install main package `docker` and `docker-compose`. The second one is needed if
you would like to use .yml files to run docker. Even though this could be
skipped, it streamlines the process.
```sh
sudo pacman -S docker
sudo pacman -S docker-compose
```

## Managing Daemon
Usually system daemon tools like systemd is used to manage daemon. Manually,
one could just run the daemon by hand, or called with runit.

```sh
sudo dockerd
```


## Check Running Docker Containers
```sh
sudo docker ps -a
```

## Start a Docker Container
```sh
sudo docker start $CONTAINER_ID
```

## Remove a Docker Container
```sh
sudo docker stop $CONTAINER_ID $CONTAINER_ID2
sudo docker rm   $CONTAINER_ID $CONTAINER_ID2
```

## How to Update Docker
### Using .yml Files (Best Though Slow)
Go to the docker-compose.yml file's directory.

Run following two commands.
```sh
sudo docker-compose pull && sudo docker-compose up -d
```

### Faster Sloppy Way
This approach might cause some bugs. I have witnessed that images names get
changed and become confusing.

```sh
sudo docker pull $CONTAINER_NAME/$CONAINER_NAME
```


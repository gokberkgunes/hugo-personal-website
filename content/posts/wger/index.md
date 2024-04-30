---
title: "Mealie Installation Steps"
date: 2023-12-27T19:58:17+03:00
authors: ["Gokberk Gunes", ]
tags: [server]
#header_imageX: ./images/title.webp
draft: true
---

## Introduction
In this guide, we will self-host a workout tracker called `wger`. `Wger` is
a free and open source software. There is a readily-available docker image that
allows quick and secure way to host the software.

## Installation Steps
Clone the docker image which is hosted on the official [git
repo](https://github.com/wger-project/docker). Note that, in this repository,
contributors suggest to pull the data from this repository frequently. These
updates are required for new configuration settings, security features and
updates.

```sh
git clone https://github.com/wger-project/docker
cd docker
```

At this point, we need to configure the settings for the docker. In order to do
this, `wger` contributors suggest to create a file called
`docker-compose.override.yml`. The main reason to creation of a new file is to
avoid complications when user updates the configuration files. Furthermore, an
example of this override file exists and named as
`docker-compose.override.example.yml`.

Create the override file.
```sh
touch docker-compose.override.yml
```

In this file, the first and most important override we should add is the
`nginx` ports. Naturally, this is needed because `wger`'s default port is
selected to be the common `80`. Since this port might be used by another
self-hosted programs, we should change the defult port's value.


```txtt
./docker-compose.override.yml
```
```txtb
services:
    nginx:
        ports:
            - "8046:80"
```

If we want to change default environmental variables we should tell where are
our environmental variables are located. We need a new file to specify the
environmental variables, default environmental variables are located at
`./config/prod.env` file. Let's create a new file here as suggested by the
contributors of `wger`.

```sh
touch ./config/my.env
```

```txtt
./config/my.env
```
```txtb
SECRET_KEY=super_random_key
SIGNING_KEY=another_super_random_key
TIME_ZONE=Europe/Istanbul

ALLOW_REGISTRATION=False
ALLOW_GUEST_USERS=False
ALLOW_UPLOAD_VIDEOS=False
MIN_ACCOUNT_AGE_TO_TRUST=0

```





Next, we simply override options to include our new environmental variables
similar to how we changed the port settings. Now, our override file should look
like below now.


```txtt
./docker-compose.override.yml
```
```txtb
services:
    nginx:
        ports:
            - "8046:80"
web:
    env_file:
        - ./config/prod.env
        - ./config/my.env
```



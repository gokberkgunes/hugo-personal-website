---
title: "Mealie Installation Steps"
date: 2023-12-27T19:58:17+03:00
authors: ["Gokberk Gunes", ]
tags: [server]
#header_imageX: ./images/title.webp
draft: true
---
doas docker search focalboard
doas docker pull mattermost/focalboard
doas docker run -d -it -p 8321:8000 --name focalboard --restart=always mattermost/focalboard

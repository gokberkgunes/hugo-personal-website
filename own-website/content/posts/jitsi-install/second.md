---
title: "Jitsi Meet Installation on Artix Linux"
date: 2022-09-28T01:40:39+03:00
draft: false
---
- Configuration files are found at
https://github.com/jitsi/jitsi-meet/tree/master/doc/debian/
- Arch wiki
https://wiki.archlinux.org/title/Jitsi-meet
- BEST GUIDE
https://blog.celogeek.com/posts/linux/archlinux/2021-02-jitsi-meet-nighly-on-arch-linux/

- Switch to root account and get required packages
```sh-root
doas su
pacman -S gnupg nginx-mainline curl java-environment-common maven npm
```

- Set local loopback so that programs can talk to each other.
/etc/hosts:
```sh
127.0.0.1 YOUR_DOMAIN auth.YOUR_DOMAIN
::1 YOUR_DOMAIN auth.YOUR_DOMAIN
```


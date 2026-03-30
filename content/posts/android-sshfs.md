---
title: "Android Sshfs"
date: 2026-03-23T00:42:49-05:00
authors: ["Gokberk Gunes", ]
tags: ["Android" ]
draft: false
---

## Intro
If you would like to sync your files between Linux and Android, you will find
out that plugging in your phone is not as versatile as using `sshfs`. There are
two reasons for this
- Android doesn't preserve metadata through USB (MTB) well.
- Delta transfer of _rsync_ doeesn't work well over USB (MTB).


Therefore, we need to figure out how to send files through ssh, or in this case
sshfs.

## SimpleSSHD
SimpleSSHD is a dropbear based ssh2 server, per its
[website](https://www.galexander.org/software/simplesshd/). We can find this
app on FDroid.

### Connection to SimpleSSHD
1. Make sure both your device and phone connected to the same modem.
2. Run SimpleSSHD.
3. Start connection from SimpleSSHD, then check phone's local IP adress through
   SimpleSSHD screen. It should be saying `... from 192.168.xxx.xxx` on the
    last line. Do not use the one saying `IP: ...`.
4. Ssh into it with SimpleSSHD's default port 2222.
    -  `ssh -p 2222 192.168.xxx.xxx`
5. Read password from phone SimpleSSHD.
6. Create `authorized_keys` file at location of ssh
    - `touch authorized_keys`
7. Add your public key there, you can use vi or redirect stdout.
    - `cat $PUBLICKEY > authorized_keys`
8. Finally run `sshfs`
    - `sshfs -F $SSH_CONFIG -p2222 user@192.168.xxx.xxx:/storage/emulated/0 /mnt/android`

### Notes
Below the quotes from the official website.

> If SimpleSSHD does not find an authorized_keys file when a client connects,
> then it generates a single-use password at that time and displays it in the
> console log. So the procedure to login the first time is to initiate the ssh
> connection, then look at the phone and type in the password that is on the
> screen in the SimpleSSHD app. It is recommended to use that shell session to
> install the authorized_keys file.
>
> Once authorized_keys exists, only public key authentication is supported. If
> you screw up your authorized_keys file, use the options menu (upper right) ->
> Reset Keys.
>
> The default home directory is now the app-private directory, which will
> generally be something like /data/data/org.galexander.sshd/files, but may vary
> depending on your device. Use options menu -> Copy App-private Path to view the
> path or copy it onto the clipboard. A nice feature of the app-private directory
> is it generally supports the full range of Unix file permissions, including
> execute! A disadvantage is that these files all disappear if you uninstall
> SimpleSSHD.
>

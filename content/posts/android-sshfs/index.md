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
1. Check phone's local IP adress through its wifi settings.
2. Run SimpleSSHD
2. Ssh into it with SimpleSSHD's default port 2222.
    -  `ssh -p 2222 192.168.xxx.xxx`
3. Read password from phone SimpleSSHD.
3. Create `authorized_keys` file at location of ssh
    - `touch authorized_keys`
4. Add your public key there, you can use vi or redirect stdout.
    - `cat $PUBLICKEY > authorized_keys`
5. Finally run `sshfs`
    - `sshfs -F $SSH_CONFIG -p2222 user@192.168.xxx.xxx:/storage/emulated/0 /mnt/storage`

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

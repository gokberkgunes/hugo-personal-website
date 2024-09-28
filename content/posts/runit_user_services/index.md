---
title: "Runit User Service Management"
date: 2024-09-28T14:41:54+03:00
authors: ["Gokberk Gunes", ]
tags: []
header_image:
draft: false
---

## Introduction 

This guide focuses on mysterious topics in GNU/Linux: user-level service
management without systemd. Joke aside, in commonplace distributions, the
widespread of systemd and its premade services allows us to skip the service
management totally. For most of us, starting applications through Xorg works
and is easy. These reasons combined totally makes us forget that we have better
ways to manage our services.

For a while, I have been using runit as my service manager. At first, runit
restricted me to set services the way I was used to because I did not know how
to use it. However, lately I got the grasp of runit and now everything seems so
natural. For example, I can connect to samba servers at the boot time whenever
my internet is up or connect to VPN servers randomly at boot time. Therefore,
I can say that my preference of runit over others is simple: it uses shell
scripts to start, stop and keep logs of the services. This means there is no
need to learn a whole system to keep services in check; also, I am free to do
whatever I want to. Surely, there are other arguments such as runit is not
being a huge piece of binary that replaces everything, yada yada.

In addition to basic setup of user service framework in runit, we will discover
how to run pipewire and mpd as a user service. Surprisingly, as of 2024-09-28,
there is no single explanation how this topic should be done online. Therefore,
the originality of this post is that we will have both pipewire and mpd working
and managed by runit.

Note that runit system-wide locations might differ based on the distrubution we
are using. Our guide will use Artix Linux' directories, i.e., `/etc/runit` and
`/run/runit/`.

## User Service Manager (Runsvdir Service) 

At this part, we will closely follow godsend documentation on user services of
Void Linux[^1]. This document also detail everything explained below though
I believe we have a little bit more clarity in this guide.

First of all, let's switch to root user and create our base runit directories.
I used the name `runsvdir` denoting the command we will call later, but you
might prefer something else like `usrsvdir` or `usersv`.
```shell
sudo -i
install -m 755 -o root -g root -d /etc/runit/sv/usersv/log
cd /etc/runit/sv/runsvdir
```

### Main Script

Now, let's create a system-wide run file that will start our user's services.
Here, we should set following variables:
- `$USER` to our username.
- `$XDG_CONFIG_HOME` to the desired location. Generally, this directory is
  gets hardcoded to ~/.config. However, here, we set it to a better location,
  ~/.local/etc, following [Earnestly's
  steps](https://github.com/Earnestly/home).
```txtt
/etc/runit/sv/usersv/run
```
```ksh
#!/usr/bin/sh

exec 2>&1
set -e
export USER="gg"
export XDG_CONFIG_HOME="/home/$USER/.local/etc"
sv_dir="$XDG_CONFIG_HOME/runit"
groups="$(id -Gn "$USER" | tr ' ' ':')"

exec chpst -u "$USER:$groups" runsvdir "$sv_dir"
```

### Creation of Logger Script

Next, we will create logging script for error tracking.

```txtt
/etc/runit/sv/usersv/log/run
```
```ksh
#!/usr/bin/sh
exec 2>&1
set -e

[ -d /var/log/runsvdir ] || install -dm 755 /var/log/runsvdir

exec svlogd -tt /var/log/runsvdir
```

We got our system-wide service ready, only step to start is symlinking it to
autorun the service.

```shell
ln -sf /etc/runit/sv/runsvdir/ /run/runit/service/
```

Now, we can start populating our `$XDG_CONFIG_HOME/runit` with our user
services. As of now, there is a slight annoyance ahead of us: we call runit as
user with `sv`, we need to give full path of the service, for example `sv start
$XDG_CONFIG_HOME/runit/pipewire`. In order to call `sv start pipewire` instead,
we should export `$SVDIR` environmental variable. Just add a such variable to
your .profile of choice.
```txtt
$XDG_CONFIG_HOME/zsh/.zprofile
```
```ksh
export SVDIR="/home/gg/.local/etc/runit/"
```

## User Services
This section of the guide will explain creation of pipewire and mpd as user
services. Along with main scripts we will have logging scripts to track errors
and outputs. Note that, unlike system-wide scripts, we do not need to create
symlink from our user scripts directory.

How to run mpd and pipewire is not clear, and since they need to communicate
with each other we need to find a way to allow this. Online, while some 
claim that pipewire must be run as inside a dbus session, others suggest we
should add server IP adresses and so on. However, the only requirement appears
as running both pipewire and mpd after exporting `$XDG_RUNTIME_DIR` in the run
script.

Before starting, let's learn where is `$XDG_RUNTIME_DIR` set in our system. For
my system, running `echo $XDG_RUNTIME_DIR` returns `/run/user/1000`. I will
also use this folder while exporting the variable. Please note that our user
services might get ran earlier than this temporary filesystem get mounted. This
incident will be found reported in the logs of our programs. Therefore, we
might also use some other folder owned by our user as `$XDG_RUNTIME_DIR`, e.g.,
`$XDG_STATE_HOME`, which is set to `/home/gg/.local/var/state` for my system.

After setting up every file, remove wireplumber, pipewire and mpd calls in your
x server. Later on, it is best to reboot the system and check if everything is
good.

For the log files, we can use the below basic file for all the user services.
We need to replace `service_name` with our wanted service name.

```txtt
$XDG_CONFIG_HOME/runit/service_name/log/run
```
```ksh
#!/usr/bin/sh
exec 2>&1
set -e

logdir=/home/gg/.local/var/log/service_name
[ -d "$logdir" ] || install -dm 755 "$logdir"
exec svlogd -tt /var/log/service_name
```

### Pipewire and Wireplumber
In addition to pipewire, we will also manage wireplumber as a user-service.
Let's prepare folders for both first.

```sh
install -m 755 -o gg -g gg -d "$SVDIR/pipewire/log"
install -m 755 -o gg -g gg -d "$SVDIR/wireplumber/log"
```
#### Pipewire Run Script

Run file for pipewire should be set as below. For the log file, use the
already-given log file template. 

```txtt
$SVDIR/wireplumber/run
```
```ksh
#!/usr/bin/sh
exec 2>&1
export XDG_RUNTIME_DIR="/run/user/1000"
exec pipewire
```
#### Wireplumber Run Script

Run file for wireplumber is almost exactly same as pipewire. Again, for the log
file we will just use the template.

```txtt
$SVDIR/pipewire/run
```
```ksh
#!/usr/bin/sh
exec 2>&1
export XDG_RUNTIME_DIR="/run/user/1000"
exec wireplumber
```


### Music Player Daemon (MPD)
Similar to pipewire, let's create folder first.
```sh
install -m 755 -o gg -g gg -d "$XDG_CONFIG_HOME/runit/mpd/log"
```
We will call mpd with verbose and in no daemon state. We would like to not run
in daemon state so that runit can control it.

```txtt
$SVDIR/mpd/run
```
```ksh
#!/usr/bin/sh
exec 2>&1
set -e
export XDG_RUNTIME_DIR="/run/user/1000"
exec /usr/bin/mpd --verbose --no-daemon
```

Lastly, we should check if mpd is using pipewire, not pulseaudio since we did
not pipewire-pulseaudio in our system. For this, we should have something like
below in our `mpd.conf`.

```txtt
$XDG_CONFIG_HOME/mpd/mpd.conf
```
```cf
audio_output {
	type  "pipewire"
	name  "mpd-pipewire"
	format  "48000:16:2" # Change 2 to 1 for mono.
}
```









[^1]: [Void Linux User Services](https://docs.voidlinux.org/config/services/user-services.html)

---
title: "Jitsi Installation on Artix Linux"
date: 2022-09-28T01:40:39+03:00
authors: "Gokberk Gunes"
tags: ["self-hosting", "server"]
draft: false
---

This article will guide you through setting up server side of a video conference
program called Jitsi on systemd-free GNU/Linux distributions. The installation
will be done on bare metal, not through any vitalization. Some commands are
given explicitly for Arch-based distributions and systems using runit as init,
though the commands can be easily altered for other init systems.

## What is Jitsi?
Jitsi is an open-source video conference software. It's similar to Zoom, Google
Meet, Microsoft Teams, except for being self-hosted. Even though the self-hosting part
generates a layer of complexion for the hosting party, the freedom and
extensibility of Jitsi are indispensable.

## Why did I write this guide?
Typically, Jitsi and some other parties provide much easier ways to
deploy Jitsi on a server. However, all of these ways require systemd, abstract
the installation process, and limit the configurability. 

**Note:** The guide is adapted from Arch Linux' wiki page, Celogreek's
guide. Yet, it still was a hassle because of the compilations of the programs and et
cetera.

Below you may find references, useful links accompany this guide.
- [Configuration files](https://github.com/jitsi/jitsi-meet/tree/master/doc/debian/)
- [Arch wiki](https://wiki.archlinux.org/title/Jitsi-meet)
- [Celogreek's guide](https://blog.celogeek.com/posts/linux/archlinux/2021-02-jitsi-meet-nighly-on-arch-linux/)

## Install Preparations

Switch to the root account. You must keep using root account throughout the whole
guide.\
**Note:** If you do not have `doas`, you should replace it with `sudo`. Alternatively, you can directly call `su`.
```sh
doas su
```

Now, you need to install the required programs. These programs are either run-time
or compile-time dependencies.
```shell
pacman -S git gnupg nginx-mainline curl jdk11-openjdk maven npm unzip prosody
```

Add the below piece of code to set a local loop back. This code will allow
programs to talk to each other.
```txtt
/etc/hosts
```
```txtb
127.0.0.1 YOUR_DOMAIN auth.YOUR_DOMAIN
::1 YOUR_DOMAIN auth.YOUR_DOMAIN
```

## Jicofo Installation

### Cloning and Installing
In this section, you will obtain the source code from official repositories,
compile the source and move them to an appropriate location.

Clone the official repository.
```shell
mkdir /srv/http/jitsi/sources && cd /srv/http/jitsi/sources
git clone "https://github.com/jitsi/jicofo.git"
cd jicofo
```

Compile the source code to executables.
```shell
mvn package -DskipTests -Dassembly.skipAssembly=false
```
Move the compiled files to the server's system folders.
```shell
unzip jicofo/target/jicofo-*.zip -d /srv/http/jitsi/
mv /srv/http/jitsi/jicofo-* /srv/http/jitsi/jicofo
rm /srv/http/jitsi/jicofo/jicofo.bat
```

### Creating and Modifying Configuration Files
Creation of the configuration files for Jicofo and their setup will be done
here. Beware that this part requires manual edits to the specified files.

Move the configuration files to a sensible location.
```shell
mkdir -p /etc/jicofo/
mv lib/logging.properties /etc/jicofo
mv jicofo-selector/src/main/resources/reference.conf /etc/jicofo/jicofo.conf
```

In the configuration file, you have to change the below sections to your domain
address. If these lines are they are commented out with either **//** or **#**,
remove these comment characters.
```txtt
/etc/jicofo/jicofo.conf
```
```txtb
xmpp {
	...
	conference-muc-jid = conference.meet.berksen.net

	...

      client-proxy = focus.meet.berksen.net

	...
    }
```

Create a user and a group for jicofo.
```shell
echo "g jitsi\nu jicofo -:jitsi - /var/lib/jicofo" > /usr/lib/sysusers.d/jicofo.conf
sysusers
```

In below file modify `JICOFO_HOSTNAME`, `JICOFO_AUTH_DOMAIN`,
`JICOFO_AUTH_PASSWORD` with your information. Change `meet.berksen.net` to your
domain address and set password1. You may generate a randomized password with
`openssl rand -hex 16`.
```txtt
/etc/jicofo/config
```
```shbot
# Jitsi Conference Focus settings
# sets the host name of the XMPP server
export JICOFO_HOST=localhost

# sets the XMPP domain (default: none)
export JICOFO_HOSTNAME=meet.berksen.net

# sets the XMPP domain name to use for XMPP user logins
export JICOFO_AUTH_DOMAIN=auth.meet.berksen.net

# sets the username to use for XMPP user logins
export JICOFO_AUTH_USER=focus

# the password to use for XMPP user logins (focus user)
export JICOFO_AUTH_PASSWORD=password1

# extra options to pass to the jicofo daemon
export JICOFO_OPTS=""

# adds java system props that are passed to jicofo (default are for home and
# logging configuration file)
export JAVA_SYS_PROPS="\
 -Dnet.java.sip.communicator.SC_HOME_DIR_LOCATION=/etc\
 -Dnet.java.sip.communicator.SC_HOME_DIR_NAME=jicofo\
 -Dnet.java.sip.communicator.SC_LOG_DIR_LOCATION=/var/log/jicofo\
 -Djava.util.logging.config.file=/etc/jicofo/logging.properties\
 -Dconfig.file=/etc/jicofo/jicofo.conf\
 -Djava.util.prefs.userRoot=/var/lib/jicofo\
"
```

Create the communicator file. Modify adress **meet.berksen.net** below.
```shell
echo "org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.meet.berksen.net" > /etc/jicofo/sip-communicator.properties
```

### Runit Script Generation
There is a need to supervise, execute, log the jicofo. Here, a runit script
will be created and activated to provide these needs.

Create a file for the jicofo. This will do the logging and running jicofo.
```shell
mkdir -p /etc/runit/sv/jicofo/log
touch /etc/runit/sv/jicofo/log/run /etc/runit/sv/jicofo/run
chmod +x /etc/runit/sv/jicofo/log/run /etc/runit/sv/jicofo/run
```

Alter the run file as shown below.
```txtt
/etc/runit/sv/jicofo/run
```
```shbot
#!/bin/sh
exec 2>&1; set -e

# Read configuration file which sets other configuration files' locations.
. /etc/jicofo/config

exec chpst -u jicofo:jitsi /srv/http/jitsi/jicofo/jicofo.sh --host=$JICOFO_HOST --domain=$JICOFO_HOSTNAME --user_name=$JICOFO_AUTH_USER --user_domain=$JICOFO_AUTH_DOMAIN $JICOFO_OPTS
```

Edit the logging script as following.
```txtt
/etc/runit/sv/jicofo/log/run
```
```shbot
#!/bin/sh
exec 2>&1; set -e

[ -d /var/log/jitsivid ] || install -dm 755 /var/log/jitsivid

exec svlogd -tt /var/log/jitsivid
```
Start the runit service.
```shell
ln -s /etc/runit/sv/jicofo /run/runit/service
```

## Jitsi-Videobridge Installation

### Cloning and Installing
At this point, the source code from official repositories will be downloaded.
The source code will be compiled. Then, compiled files will be moved to an
appropriate location.

Clone the repository.
```shell
cd /srv/http/jitsi/sources/
git clone "https://github.com/jitsi/jitsi-videobridge.git"
cd jitsi-videobridge
```
Compile jitsi-videobridge.
```shell
mvn package -DskipTests -Dassembly.skipAssembly=true install
```
Move the compiled files to the server's system folders.
```shell
unzip jvb/target/jitsi-videobridge-*.zip -d /srv/http/jitsi/
mv /srv/http/jitsi/jitsi-videobridge-* /srv/http/jitsi/jitsi-videobridge
rm /srv/http/jitsi/jitsi-videobridge/jvb.bat
```
### Creating and Modifying Configuration Files
Configuration files for jitsi-videobridge will be done here. Please note that
there are parts you need to edit manually. Guidance is given on what and how
to edit.

Move the java configuration and kernel setting files to a better location.
```shell
mkdir -p /etc/jitsi-videobridge/
mv jvb/lib/logging.properties config/callstats-java-sdk.properties /etc/jitsi-videobridge
mkdir -p /etc/sysctl.d
mv config/20-jvb-udp-buffers.conf /etc/sysctl.d
```

Create a user and a group for jitsi-videobridge.
```shell
echo "g jitsi\nu jvb -:jitsi - /var/lib/jitsi-videobridge" > /usr/lib/sysusers.d/jitsi.conf
sysusers
```

Modify `JVB_HOSTNAME` to your domain; in other words alter `meet.berksen.net`
reflecting your domain. Set `password2`. You may generate a randomized
password with `opensll rand -hex 16`\
**Note:** The configuration file could have been left empty at this point; the
server would still be working. However, it will be needed if a web socket is
used.

```txtt
/etc/jitsi-videobridge/config
```
```shbot
# Jitsi Videobridge settings

# sets the XMPP domain (default: none)
export JVB_HOSTNAME=meet.berksen.net

# sets the hostname of the XMPP server (default: domain if set, localhost otherwise)
export JVB_HOST=

# sets the port of the XMPP server (default: 5275)
export JVB_PORT=5347

# shared secret used to authenticate to the XMPP server (jvb user)
export JVB_SECRET=password2

# extra options to pass to the JVB daemon
export JVB_OPTS="--apis=,"

# adds java system props that are passed to jvb (default are for home and logging configuration file)
export JAVA_SYS_PROPS="\
  -Dnet.java.sip.communicator.SC_HOME_DIR_LOCATION=/etc\
  -Dnet.java.sip.communicator.SC_HOME_DIR_NAME=jitsi-videobridge\
  -Dnet.java.sip.communicator.SC_LOG_DIR_LOCATION=/var/log/jitsi-videobridge\
  -Djava.util.logging.config.file=/etc/jitsi-videobridge/logging.properties\
  -Djava.util.prefs.userRoot=/var/lib/jitsi-videobridge\
"
# Below configuration belongs to JAVA_SYS_PROPS, but causes jitsi to not work.
#  -Dconfig.file=/etc/jitsi-videobridge/jvb.conf\
```

Create the communicator file. Change `shard.DOMAIN` to your domain. Set
`password2` as what you set earlier. Modify domain part `shard.MUC_JIDS` to your
domain. Generate a `Shard.MUC_NICKNAME` with `uuidgen` command.
```txtt
/etc/jitsi-videobridge/sip-communicator.properties
```
```txtb
org.ice4j.ice.harvest.DISABLE_AWS_HARVESTER=true
org.ice4j.ice.harvest.STUN_MAPPING_HARVESTER_ADDRESSES=meet-jit-si-turnrelay.jitsi.net:443
org.jitsi.videobridge.ENABLE_STATISTICS=false
org.jitsi.videobridge.STATISTICS_TRANSPORT=muc
org.jitsi.videobridge.xmpp.user.shard.HOSTNAME=localhost
org.jitsi.videobridge.xmpp.user.shard.DOMAIN=auth.meet.berksen.net
org.jitsi.videobridge.xmpp.user.shard.USERNAME=jvb
org.jitsi.videobridge.xmpp.user.shard.PASSWORD=password2
org.jitsi.videobridge.xmpp.user.shard.MUC_JIDS=JvbBrewery@internal.auth.meet.berksen.net
org.jitsi.videobridge.xmpp.user.shard.MUC_NICKNAME=uuid
```

### Runit Script Generation
There is a need to supervise, execute, log the jitsi-videobridge. Here, a runit
script will be created and activated to provide these needs.

Create a file for the jitsi-videobridge. This will do the logging and running jitsi-videobridge.
```shell
mkdir -p /etc/runit/sv/jitsi-videobridge/log
touch /etc/runit/sv/jitsi-videobridge/log/run /etc/runit/sv/jitsi-videobridge/run
chmod +x /etc/runit/sv/jitsi-videobridge/log/run /etc/runit/sv/jitsi-videobridge/run
```

Alter the run file as shown below.
```txtt
/etc/runit/sv/jitsi-videobridge/run
```
```shbot
#!/bin/sh
exec 2>&1; set -e


# Read configuration file which sets other configuration files' locations.
. /etc/jitsi-videobridge/config

exec chpst -u jvb:jitsi /srv/http/jitsi/jitsi-videobridge/jvb.sh $JVB_OPTS
```

Edit the logging script as following.
```txtt
/etc/runit/sv/jitsi-videobridge/log/run
```
```shbot
#!/bin/sh
exec 2>&1; set -e

[ -d /var/log/jitsivid ] || install -dm 755 /var/log/jitsivid

exec svlogd -tt /var/log/jitsivid
```

Start the runit service.
```shell
ln -s /etc/runit/sv/jitsi-videobridge /run/runit/service
```

## Jitsi-Meet Installation
### Cloning and Installing
In this section, the source code from official repositories will be downloaded.
The source code will be compiled. Then, compiled files will be moved to an
appropriate location.

Clone the repository.
```shell
mkdir -p /srv/http/jitsi/; cd /srv/http/jitsi
git clone "https://github.com/jitsi/jitsi-meet.git"
cd jitsi-meet/
```

Compile jicofo.\
**Note:** `make` command for jitsi-meet will not work cause of ram limitations
in java settings. That's why you are exporting `NODE_OPTIONS` environmental
variable as shown below.
```shell
export NODE_OPTIONS="--max-old-space-size=8192"
npm install
make
```

### Creating and Modifying Configuration Files
Configuration files for jitsi-meet will be done here. Please note that there
are parts you need to edit manually. Guidance is given on what and how to
edit.

Move the compiled files to the server's system folders.\
**Note:** There used to be a configuration file called **logging_config.js** but
it is now merged into **config.js**. The other file, **interface_config.js**,
is also planned to be merged into **config.js** in the future.
```shell
mkdir -p /etc/jitsi-meet
ln -sf /srv/http/jitsi/jitsi-meet/interface_config.js /etc/jitsi-meet/
ln -sf /srv/http/jitsi/jitsi-meet/config.js /etc/jitsi-meet/
```

Edit the main configuration file. Change below options for your own domain.
```txtt
/etc/jitsi-meet/config.js
```
```txtb
var config = {
    ...
	hosts: {
		// XMPP domain.
		domain: 'meet.berksen.net',

		...
		// XMPP MUC domain. FIXME: use XEP-0030 to discover it.
		muc: 'conference.meet.berksen.net',
	},

	// BOSH URL. FIXME: use XEP-0156 to discover it.
	bosh: '//meet.berksen.net/http-bind',

	...
}
```

## Prosody Configuration
Prosody is found in repositories as in pre-compiled form. Therefore, there is
only configuration is required for it.

### Creating and Modifying Configuration Files
Make a configuration folder. Download configuration file from jitsi-meet
repository. Include the configuration file to prosody's settings.
```shell
mkdir -p /etc/prosody/conf.d
curl "https://raw.githubusercontent.com/jitsi/jitsi-meet/master/doc/debian/jitsi-meet-prosody/prosody.cfg.lua-jvb.example" > /etc/prosody/conf.d/jitsi.cfg.lua
echo "Include \"conf.d/*.cfg.lua\"" >> /etc/prosody/prosody.cfg.lua
```
Change domain names to your domain as described below. In this file, you should
replace all domain names with yours. (If desired, all `jitmeet.example.com`
occurrences can be changed to your domain too.)
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
Replace all occurances:
    "jitmeet.example.com" with "meet.berksen.net"
    "focusUser@auth.jitmeet.example.com" with "focus@auth.meet.berksen.net"
Vim users can run following command:
:%s/jitmeet\.example\.com/meet\.berksen\.net/g
```

Set plugin paths as shown below.
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
plugin_paths = { "/srv/http/jitsi/jitsi-meet/resources/prosody-plugins/" }
```

### Certificate Generation
In order to generate certificates, you muust have CNAME records by your domain
provider. There are going to be some warnings about certificate for hosts. The
warnings are not important since all warnings are about virtual hosts, which
you do not need certificates for. Only meet.berksen.net and
auth.meet.berksen.net are important.
```sh
certbot --nginx certonly -d meet.berksen.net
certbot --nginx certonly -d auth.meet.berksen.net
```
Create a letsencrypt renewal hook. This way, whenever you renew the letsencrypt
certificates, prosody certificates will automatically imported.
```txtt
/etc/letsencrypt/renewal-hooks/deploy/prosody
```
```shbot
#!/usr/bin/sh
/usr/bin/prosodyctl --root cert import /etc/letsencrypt/live
```

Make the hook file executable.
```sh
chmod +x /etc/letsencrypt/renewal-hooks/deploy/prosody
```

After running `certbot`, you will get configuration and certificate files.
- Configuration is written to `/var/lib/prosody/meet.berksen.net.cnf`,
- Certificate is written to `/var/lib/prosody/meet.berksen.net.crt`.

Now change keys and certificates, as indicated above.
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
VirtualHost "meet.berksen.net"
	...
	ssl = {
		key = "/etc/prosody/certs/meet.berksen.net.key";
		certificate = "/etc/prosody/certs/meet.berksen.net.crt";
	}

	...

VirtualHost "auth.meet.berksen.net"
	ssl = {
		key = "/etc/prosody/certs/auth.meet.berksen.net.key";
		certificate = "/etc/prosody/certs/auth.meet.berksen.net.crt";
	}
	authentication = "internal_hashed"
```

## Nginx & Jitsi Connection
Make a configuration folder and a configuration file as **jitsi.conf**.
```sh
mkdir -p /etc/nginx/enabled-sites
mkdir -p /etc/nginx/sites
curl "https://github.com/jitsi/jitsi-meet/blob/master/doc/debian/jitsi-meet/jitsi-meet.example" > /etc/nginx/sites
mv /etc/nginx/sites/jitsi-meet.example /etc/nginx/sites/jitsi.conf
```
Activate the configuration file in nginx by symlinking it.
``` sh
ln -sf /etc/nginx/sites/jitsi.conf /etc/nginx/enabled-sites
```
Modify nginx' configuration file to include all the configuration files located
in enabled-sites folder. The include statement must go inside the http part.
```txtt
/etc/nginx/nginx.conf
```
```txtb
http {

	...

	include /etc/nginx/enabled-sites/*.conf;

	...

}
```

Alter **jitsi.conf** to include your domain names, SSL certificates,
installation file paths.
```txtt
/etc/nginx/enabled-sites/jitsi.conf
```
```txtb
server {
	...

	server_name meet.berksen.net;

	...

	ssl_certificate /etc/prosody/certs/meet.berksen.net.crt;
	ssl_certificate_key /etc/prosody/certs/meet.berksen.net.key;

	root /srv/http/jitsi/jitsi-meet;

	location = /config.js {
		alias /srv/http/jitsi/jitsi-meet/config.js;
	}

	location = /external_api.js {
		alias /srv/http/jitsi/jitsi-meet/libs/external_api.min.js;
	}

	...

	# ensure all static content can always be found first
	location ~ ^/(libs|css|static|images|fonts|lang|sounds|connection_optimization|.well-known)/(.*)$
	{
		alias /srv/http/jitsi/jitsi-meet/$1/$2;

		...
	}

	...

	location ~ ^/([^/?&:'"]+)/config.js$
	{

		...
		alias /srv/http/jitsi/jitsi-meet/config.js;
	}
```

Run the following command to check syntax errors in nginx configuration file.
```sh
nginx -t
```

## Register Users
Register for jicofo. You may check your password1 in `/etc/jicofo/config`.
```sh
prosodyctl register focus auth.meet.berksen.net password1
```
Register for jitsi-videobridge. You may check your password2 at `/etc/jitsi-videobridge/config`.
```sh
prosodyctl register jvb auth.meet.berksen.net password2
```
Subscribe to mod_roster_command.
```sh
prosodyctl mod_roster_command subscribe focus.meet.berksen.net focus@auth.meet.berksen.net
```
Restart runit services.
```sh
sv restart prosody
sv restart jitsi-videobridge
sv restart jicofo
sv restart nginx
```


## Websocket Activations

This part of the installation is done in order to decrease server's load.

### Jitsi-Meet Websocket Activation
Activate websocket on jitsi-meet side.
 ```txtt
/etc/jitsi-meet/config.js
```
```txtb
var config = {
	...

	// Websocket URL
	websocket: 'wss://meet.berksen.net/xmpp-websocket',

	...
}
```

### Prosody Websocket Activation
Activate the websocket module on prosody configuration file.
 ```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
consider_websocket_secure = true;
cross_domain_websocket = true;

VirtualHost "meet.berksen.net"
	...

	modules_enabled = {
		...
		"websocket"; -- Enable websocket.
		...
	}
```

### Jitsi-Videobridge Websocket Activation
Copy the reference jitsi-videobridge configuration file to your system.
```sh
cp /srv/http/jitsi/sources/jitsi-videobridge/jvb/src/main/resources/reference.conf /etc/jitsi-videobridge/
```

Set sockets on the jitsi-videobridge.
```txtt
/etc/jitsi-videobridge/reference.conf
```
```txtb
videobridge {
	http-servers {
		...
		public {
			...
			port = 9090
			...
		}
		...
	}
	...
	websockets {
		...
		enabled=true
		server-id="default-id"
		tls=true
		domain="meet.berksen.net:443"
		...
	}
}
```

Add socket info back to the jitsi-videobridge configuration file.
```txtt
/etc/jitsi-videobridge/config
```
```shbot
...
# adds java system props that are passed to jvb (default are for home and logging configuration file)
export JAVA_SYS_PROPS="\
  -Dnet.java.sip.communicator.SC_HOME_DIR_LOCATION=/etc\
  -Dnet.java.sip.communicator.SC_HOME_DIR_NAME=jitsi-videobridge\
  -Dconfig.file=/etc/jitsi-videobridge/reference.conf\
  -Dnet.java.sip.communicator.SC_LOG_DIR_LOCATION=/var/log/jitsi-videobridge\
  -Djava.util.logging.config.file=/etc/jitsi-videobridge/logging.properties\
  -Djava.util.prefs.userRoot=/var/lib/jitsi-videobridge\
"
```

Restart the services.
```sh
sv restart prosody
sv restart jitsi-videobridge
```

## Disallow Room Creations

The rationale is to stop random people to create and use rooms, this server is
not free to use. When there is no administrator is online, other users has to
wait for an administrator. These users cannot create or use a room without an
administrator.

Add below to jicofo's sip communicator file, use correct domain name.
```txtt
/etc/jicofo/sip-communicator.properties
```
```txtb
org.jitsi.jicofo.auth.URL=XMPP:meet.berksen.net
```

Alter the jitsi-meet's configuration file to reflect your domain name.
```txtt
/etc/jitsi-meet/config.js
```
```txtb
var config = {
	...
	host: {
		...
		// When using authentication, domain for guest users.
		anonymousdomain: 'guest.meet.berksen.net',
		...
	},
	...
}
```

Modify jitsi's prosody configuration file. Change authentification for main
domain, add guest domain with anonymous authentication. Use same modules you
use for your main virtualhost but remove muc_lobby_rooms from it.
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
...
VirtualHost "meet.berksen.net"
	...
	authentification = "internal_plain"

-- add guest virtual host to allow anonymous user to join your room
VirtualHost "guest.meet.berksen.net"
        authentication = "anonymous"
        c2s_require_encryption = false
	modules_enabled = {
		"bosh";
		"pubsub";
		"ping"; -- Enable mod_ping
		"websocket"; -- Enable websocket.
		"speakerstats";
		"external_services";
		"conference_duration";
		"muc_breakout_rooms";
		"av_moderation";
    }
    ...
```
Now, register users as many as required. The `user_name` and `user_password`
will be used to log into the server.
```sh
prosodyctl register user_name meet.berksen.net user_password
```

**NOTE**: If you need to check the current prosody users and their passwords,
go to `/var/lib/prosody`.

Restart services.
```sh
sv restart jicofo
sv restart prosody
```

You may check current accounts and their passwords below.
```txtt
/var/lib/prosody/domain_name/accounts/name.dat
```
```txtb
return {
	["password"] = "account_password";
};
```


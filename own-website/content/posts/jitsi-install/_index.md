---
title: "Jitsi Installation on Artix Linux"
date: 2022-09-28T01:40:39+03:00
draft: false
---
This article will guide one to set up video conference website called Jitsi.
Jitsi is an open-source and an ethical piece of software. It must be preferred
to spyware programs like Zoom, Google Meet, Microsoft Teams and such proprietary
alternatives. All of these malicious programs got popularized and enforced by
people who has no idea about technology. Consequently, for the people with
freedom and security in mind, Jitsi is the prime service to use and self-host.
This guide will cover the self-hosting part, usage is piece of cake anyway.
(The guide is adapted from Arch Linux' wiki page and Celogreek's guide.)


Below one may find references, useful links to accompany this guide.
- [Configuration files](https://github.com/jitsi/jitsi-meet/tree/master/doc/debian/)
- [Arch wiki](https://wiki.archlinux.org/title/Jitsi-meet)
- [Celogreek's guide](https://blog.celogeek.com/posts/linux/archlinux/2021-02-jitsi-meet-nighly-on-arch-linux/)

Switch to root account, keep using root throughout the whole guide. (Use sudo
or su preferably.)
```sh
doas su
```
Install required programs.
```shell
pacman -S git gnupg nginx-mainline curl jdk11-openjdk maven npm unzip prosody
```


Append two lines of code to set a local loopback.  The programs we are going to
install will talk to each other with local loopback.
```txtt
/etc/hosts
```
```txtb
127.0.0.1 YOUR_DOMAIN auth.YOUR_DOMAIN
::1 YOUR_DOMAIN auth.YOUR_DOMAIN
```

## Jicofo Installation

### Cloning and Installing Jicofo
Clone the repository from official repositories, then compile Jicofo.
```shell
mkdir /srv/http/jitsi/sources && cd /srv/http/jitsi/sources
git clone "https://github.com/jitsi/jicofo.git"
cd jicofo
mvn package -DskipTests -Dassembly.skipAssembly=false
```

 After compilation is done move the compiled files to your server's system
 folders.
```shell
unzip jicofo/target/jicofo-*.zip -d /srv/http/jitsi/
mv /srv/http/jitsi/jicofo-* /srv/http/jitsi/jicofo
rm /srv/http/jitsi/jicofo/jicofo.bat
```

### Creating and Modifying Configuration Files

Move configuration files to system folders.
```shell
mkdir -p /etc/jicofo/
mv lib/logging.properties /etc/jicofo
mv jicofo-selector/src/main/resources/reference.conf /etc/jicofo/jicofo.conf
```

In configuration file of jicofo change below sections to your domain adress. If
they are commented out with either **//** or **#** remove it.
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


Create a user and group for jicofo.
```shell
echo "g jitsi\nu jicofo -:jitsi - /var/lib/jicofo" > /usr/lib/sysusers.d/jicofo.conf
sysusers
```

In below file modify **JICOFO_HOSTNAME**, **JICOFO_AUTH_DOMAIN**,
**JICOFO_AUTH_PASSWORD** with your information.

Change **meet.berksen.net** to your domain adress and change password to your
focus user's password. For the password, the one may use `openssl rand -hex 16`.
```txtt
/etc/jicofo/config
```
```txtb
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
# logging config file)
export JAVA_SYS_PROPS="\
 -Dnet.java.sip.communicator.SC_HOME_DIR_LOCATION=/etc\
 -Dnet.java.sip.communicator.SC_HOME_DIR_NAME=jicofo\
 -Dnet.java.sip.communicator.SC_LOG_DIR_LOCATION=/var/log/jicofo\
 -Djava.util.logging.config.file=/etc/jicofo/logging.properties\
 -Dconfig.file=/etc/jicofo/jicofo.conf\
 -Djava.util.prefs.userRoot=/var/lib/jicofo\
"
```

Create the communicator file. Change **meet.berksen.net** to your address.
```shell
echo "org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.meet.berksen.net" > /etc/jicofo/sip-communicator.properties
```

### Runit Script Generation

Create a runit script files for the jitsi-videobridge for logging and running.
```shell
mkdir -p /etc/runit/sv/jicofo/log
touch /etc/runit/sv/jicofo/log/run /etc/runit/sv/jicofo/run
chmod +x /etc/runit/sv/jicofo/log/run /etc/runit/sv/jicofo/run
```

Edit service script.
```txtt
/etc/runit/sv/jicofo/run
```
```txtb
#!/bin/sh
exec 2>&1; set -e

# Read config file whichsets other configuration files' locations.
. /etc/jicofo/config

exec chpst -u jicofo:jitsi /srv/http/jitsi/jicofo/jicofo.sh --host=$JICOFO_HOST --domain=$JICOFO_HOSTNAME --user_name=$JICOFO_AUTH_USER --user_domain=$JICOFO_AUTH_DOMAIN $JICOFO_OPTS
```


Edit logging script.
```txtt
/etc/runit/sv/jicofo/log/run
```
```txtb
#!/bin/sh
exec 2>&1; set -e

[ -d /var/log/jitsivid ] || install -dm 755 /var/log/jitsivid

exec svlogd -tt /var/log/jitsivid
```

Start runit service.
```shell
ln -s /etc/runit/sv/jicofo /run/runit/service
```

## Jitsi Videobridge Installation
Clone the repository and compile jitsi-videobridge.
```shell
cd /srv/http/jitsi/sources/
git clone "https://github.com/jitsi/jitsi-videobridge.git"
cd jitsi-videobridge
mvn package -DskipTests -Dassembly.skipAssembly=true install
```

After compilation is done move the compiled files to your server's location.
```shell
unzip jvb/target/jitsi-videobridge-*.zip -d /srv/http/jitsi/
mv /srv/http/jitsi/jitsi-videobridge-* /srv/http/jitsi/jitsi-videobridge
rm /srv/http/jitsi/jitsi-videobridge/jvb.bat
```

Move java configuration file and then kernel setting file.
```shell
mkdir -p /etc/jitsi-videobridge/
mv jvb/lib/logging.properties config/callstats-java-sdk.properties /etc/jitsi-videobridge
mkdir -p /etc/sysctl.d
mv config/20-jvb-udp-buffers.conf /etc/sysctl.d
```

Create a user and group for jitsi-videobridge.
```shell
echo "g jitsi\nu jvb -:jitsi - /var/lib/jitsi-videobridge" > /usr/lib/sysusers.d/jitsi.conf
sysusers
```

Create config file and fill **JVB_HOSTNAME**. Change **meet.berksen.net**
to your domain. Use `opensll rand -hex 16` for **JVB_SECRET**.

Note that, config file should be empty at this point to have a working server.
This file will be used for socket later on.

```txtt
/etc/jitsi-videobridge/config
```
```txtb
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

# adds java system props that are passed to jvb (default are for home and logging config file)
export JAVA_SYS_PROPS="\
  -Dnet.java.sip.communicator.SC_HOME_DIR_LOCATION=/etc\
  -Dnet.java.sip.communicator.SC_HOME_DIR_NAME=jitsi-videobridge\
  -Dnet.java.sip.communicator.SC_LOG_DIR_LOCATION=/var/log/jitsi-videobridge\
  -Djava.util.logging.config.file=/etc/jitsi-videobridge/logging.properties\
  -Djava.util.prefs.userRoot=/var/lib/jitsi-videobridge\
"
# Below config belongs to JAVA_SYS_PROPS, but causes jitsi to not work.
#  -Dconfig.file=/etc/jitsi-videobridge/jvb.conf\
```

Create communicator file. Change **shard.DOMAIN**, **shard.PASSWORD** as jvb password
which is set on config file above as **JVB_SECRET**, lastly modify
**shard.MUC_JIDS**. **Shard.MUC_NICKNAME** will be generated with `uuidgen`
command.
```txtt
/etc/jitsi-videobridge/sip-communicator.properties
```
```txtb
org.ice4j.ice.harvest.DISABLE_AWS_HARVESTER=true
org.ice4j.ice.harvest.STUN_MAPPING_HARVESTER_ADDRESSES=meet-jit-si-turnrelay.jitsi.net:443
org.jitsi.videobridge.ENABLE_STATISTICS=true
org.jitsi.videobridge.STATISTICS_TRANSPORT=muc
org.jitsi.videobridge.xmpp.user.shard.HOSTNAME=localhost
org.jitsi.videobridge.xmpp.user.shard.DOMAIN=auth.meet.berksen.net
org.jitsi.videobridge.xmpp.user.shard.USERNAME=jvb
org.jitsi.videobridge.xmpp.user.shard.PASSWORD=password
org.jitsi.videobridge.xmpp.user.shard.MUC_JIDS=JvbBrewery@internal.auth.meet.berksen.net
org.jitsi.videobridge.xmpp.user.shard.MUC_NICKNAME=uuid
```


### Runit Script Generation

Create a runit script for the jitsi-videobridge.
```shell
mkdir -p /etc/runit/sv/jitsi-videobridge/log
touch /etc/runit/sv/jitsi-videobridge/log/run /etc/runit/sv/jitsi-videobridge/run
chmod +x /etc/runit/sv/jitsi-videobridge/log/run /etc/runit/sv/jitsi-videobridge/run
```

Edit service script.
```txtt
/etc/runit/sv/jitsi-videobridge/run
```
```txtb
#!/bin/sh
exec 2>&1; set -e


# Read config file whichsets other configuration files' locations.
. /etc/jitsi-videobridge/config

exec chpst -u jvb:jitsi /srv/http/jitsi/jitsi-videobridge/jvb.sh $JVB_OPTS
```

Edit logging script.
```txtt
/etc/runit/sv/jitsi-videobridge/log/run
```
```txtb
#!/bin/sh
exec 2>&1; set -e

[ -d /var/log/jitsivid ] || install -dm 755 /var/log/jitsivid

exec svlogd -tt /var/log/jitsivid
```

Start runit service.
```shell
ln -s /etc/runit/sv/jitsi-videobridge /run/runit/service
```

## Jitsi-Meet Installation

Clone the repository and compile jicofo. Make command for jitsi-meet will not
work cause of ram limitations in java settings. Then one has to change it with
*NODE_OPTIONS* environmental variable as shown below.
```shell
mkdir -p /srv/http/jitsi/; cd /srv/http/jitsi
git clone "https://github.com/jitsi/jitsi-meet.git"
cd jitsi-meet/
export NODE_OPTIONS="--max-old-space-size=8192"
npm install
make
```

After compilation is done symlink the compiled files to reachable location.
Note that there used to be configuration file called **logging_config.js**, it is
now merged into **config.js**. In the future **interface_config.js** is going
to merge into **config.js** as well.

```shell
mkdir -p /etc/jitsi-meet
ln -sf /srv/http/jitsi/jitsi-meet/interface_config.js /etc/jitsi-meet/
ln -sf /srv/http/jitsi/jitsi-meet/config.js /etc/jitsi-meet/
```

- Edit main configuration file, find below options and alter them for your own
  domain.
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

Make a config folder and config file and include config file to settings.
```shell
mkdir -p /etc/prosody/conf.d
curl "https://raw.githubusercontent.com/jitsi/jitsi-meet/master/doc/debian/jitsi-meet-prosody/prosody.cfg.lua-jvb.example" > /etc/prosody/conf.d/jitsi.cfg.lua
echo "Include \"conf.d/*.cfg.lua\"" >> /etc/prosody/prosody.cfg.lua
```

Change domain names to your domain as described below.
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
Replace all:

    "jitmeet.example.com" with "meet.berksen.net"
    "focusUser@auth.meet.berksen.net" with "focus@auth.meet.berksen.net"
```

Then adjust the configuration at your needs.
```txtt
/etc/prosody/conf.d/jitsi.cfg.lua
```
```txtb
plugin_paths = { "/srv/http/jitsi/jitsi-meet/resources/prosody-plugins/" }
```


Generate certificates. (Must have CNAME records by DNS). There are going to be
some warnings about certificate for hosts. These are not important since these
are all virtual hosts. Only meet.berksen.net and auth.meet.berksen.net are
important.
```sh
certbot --nginx certonly -d meet.berksen.net
certbot --nginx certonly -d auth.meet.berksen.net
```
Create a letsencrypt renewal hook. This way, whenever one renews letsencrypt
certificates, prosody certificates will automatically imported.
```txtt
/etc/letsencrypt/renewal-hooks/deploy/prosody
```
```txtb
#!/usr/bin/sh
/usr/bin/prosodyctl --root cert import /etc/letsencrypt/live
```

Make the hook file executable.
```sh
chmod +x /etc/letsencrypt/renewal-hooks/deploy/prosody

```

- Config written to /var/lib/prosody/meet.berksen.net.cnf
- Certificate written to /var/lib/prosody/meet.berksen.net.crt

Now set keys and certificates, add these below.
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

Make a config folder and config file as **jitsi.conf**.
```sh
mkdir -p /etc/nginx/enabled-sites
mkdir -p /etc/nginx/sites
curl "https://github.com/jitsi/jitsi-meet/blob/master/doc/debian/jitsi-meet/jitsi-meet.example" > /etc/nginx/sites
mv /etc/nginx/sites/jitsi-meet.example /etc/nginx/sites/jitsi.conf
ln -sf /etc/nginx/sites/jitsi.conf /etc/nginx/enabled-sites
```

Modify nginx's configuration file to include all the configuration files
located in sites folder. This include statement should go in to the http part.
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

Check if any problem exists with the nginx's configuration file.
```sh
nginx -t
```
## Register Users

Register for jicofo. (Check password at /etc/jicofo/config)
```sh
prosodyctl register focus auth.meet.berksen.net password1
```
Register for jitsi-videobridge. (Check password at /etc/jitsi-videobridge/config)
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

**NOTE**: To check current jicofo users refer to https://github.com/jitsi/jitsi-meet/issues/5625



## Activate Websockets

This part of the installation is done so that the server's load could decrease
and the Jitsi process get faster.

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

 Activate websocket module on prosody configuration file.
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

Copy the reference configuration file to the system.
```sh
cp /srv/http/jitsi/sources/jitsi-videobridge/jvb/src/main/resources/reference.conf /etc/jitsi-videobridge/
```

Set sockets on jitsi-videobridge.
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

Add socket info to jitsi-videobridge config file back.
```txtt
/etc/jitsi-videobridge/config
```
```txtb
...
# adds java system props that are passed to jvb (default are for home and logging config file)
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
not free to use.

- Add below to jicofo's sip communicator file, use correct domain name.
```txtt
/etc/jicofo/sip-communicator.properties
```
```txtb
org.jitsi.jicofo.auth.URL=XMPP:meet.berksen.net
```

Alter jitsi-meet's configuration file for your domain names.
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
Now, register users as many as required.
```sh
prosodyctl register user_name meet.berksen.net password_here
```

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


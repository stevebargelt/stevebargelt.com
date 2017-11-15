+++
aliases      = []
categories   = []
date         = "2017-09-23T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = true
slug         = ""
tags         = []
title        = "Running HomeAssistant on a RPi in Docker"
type         = "post"
weight       = 0
+++

See [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose as we will be building on that foundation to get Home Assistant set up.

### Home Assistant

If you are interested in home automation and you are not familiar with [Home Assistant](https://home-assistant.io) you should be. I haven't been so excited about home automation since I first dabbled many years ago with X-10 switches! HA is an open source project authored in Python. The community is huge and I've yet to run into a problem that I could not solve by searching the forums or the web. So, why am I writing this blog series? I think my setup is fairly typical of a power user of HA. MQTT (*2 - internal and external), Dasher, Appdaemon, Influx DB, Grafana, SSL, custom domain, and HA (of course). I'm also running everything in Docker containers on my Raspberry Pi.

```shell
docker ps --format "table {{.Image}}\t{{.Names}}"

IMAGE                                        NAMES
ha_appdaemon                                 ha_appdaemon_1
ha_dasher                                    ha_dasher_1
homeassistant/raspberrypi3-homeassistant     ha_home-assistant_1
mosquitto-rpi                                ha_mqtt-int_1
pithings/rpi-grafana                         ha_grafana_1
mosquitto-rpi                                ha_mqtt-ext_1
hypriot/rpi-influxdb                         ha_influxdb_1
```

Although I could find the answers to all of my questions and roadblocks along the way I wanted to put together a comprehensive guide that may save some pain and head-scratching for others.

In this post we will simply get Home Assistant up and running on our Raspberry Pi in a Docker container, start our Docker Compose file, and setup a development pipeline to keep our configuration up to date in Github with testing in Travis CI.

### 

First, we will SSH into our Raspberry Pi. If your Pi isn't setup for this see [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}). (and obviously use *your* IP address!)

```shell
ssh pi@192.168.1.6
```
<!-- Clone the first part of the series from the branch blog001:

```shell
git clone -b blog001 https://github.com/stevebargelt/home-assistant-config.git homeassistant
``` -->
We will create our docker-compose file:

```shell
mkdir homeassistant
cd homeassistant
nano docker-compose.yml
```

The content of that file should be:

```yaml
version: '3.1'
services:
  home-assistant:
    image: homeassistant/raspberrypi3-homeassistant
    restart: always
    volumes:
      - /home/pi/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    network_mode: "host"
```

CTRL-X, Y, Enter to save the file

Next we will create a very basic configuration file.

```shell
nano configuration.yaml
```

```yaml
homeassistant:
  name: Bargelt Home
  latitude: !secret latitude
  longitude: !secret longitude
  elevation: 273
  unit_system: imperial
  time_zone: America/Chicago

http:
  api_password: !secret api_password
  login_attempts_threshold: 3

frontend:
config:
updater:
discovery:
sun:
logger:
  default: warn
```

CTRL-X, Y, Enter to save the file

No More Secrets. Actually we want a lot of secrets. You will need to create a secrets.yaml file

```shell
nano secrets.yaml
```

You will, at the very least want to add a password for HA:

```yaml
api_password: correcthorsebatterystaple
latitude: 00.000000
longitude: 00.000000
```

CTRL-X, Y, Enter to save the file and then we can finally startup the Home Assistant container:

```shell
docker-compose -p ha up -d
```

The first time you do this step it will take quite some time 5-10 minutes or more.

```shell
docker ps
```

You should see your container running. You can see the Home Assistant logs:

```shell
docker logs ha_home-assistant_1
```

In a browser: http://<your IP address>:8123 in my case that is http://192.168.1.6:8123

You should see the Home Assistant html interface! Congratulations, you have HA running in a Docker container on your Raspberry Pi!

### Makefile

The next thing I do when creating a project is to create a makefile.

```shell
nano makefile
```

```make
build:
	@docker-compose -p ha build
run:
	@docker-compose -p ha up -d
stop:
	@docker-compose -p ha stop
clean:	stop
	@docker-compose -p ha rm home-assistant
clean-images:
	@docker rmi 'docker images -q -f "dangling=true"'
```

The makefile is a good way to learn the docker commands we will use during this series.

```shell
make run
```

is the equivalent of

```shell
docker-compose -p ha up -d
```

Using the makefile is also an easy way to **forget** the docker commands. Use if you wish.


### Tailing the logs

```shell
docker logs --tail="200" -f ha_home-assistant_1
```
CTRL-C to exit

You will use this often. Home Assistant logs are very useful for debugging and trouble shooting your HA configuration. 

### Git and Github

In [the first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) there were two optional steps. One was setting up your SSH keys to allow you to SSH from the Raspberry Pi. This is useful for pulling and pushing from git. If you didn't complete that option step - go back now and do that.

If you are using github you can use the web interface to create a new repo then back on your Raspberry Pi...in your homeassistant directory:

```shell
nano .gitignore
```

The contents of .gitignore will be

```text
*.pid
*.xml
*.csr
*.crt
*.key
icloud
www
OZW_Log.txt
home-assistant.log
home-assistant_v2.db
*.db-journal
lib
deps
tts
secrets.yaml
known_devices.yaml
phue.conf
harmony_media_room.conf
pyozw.sqlite
!/.gitignore
.ios.conf
.HA_VERSION
pwfile
.uuid
mosquitto-int.conf
plex.conf
mysqlrootpassword.txt
mysqlpassword.txt
/mysql
.vscode
.DS_Store
/dasher/config
appdaemon/*
!appdaemon/Dockerfile
appdaemon/conf
```

```shell
git init
git add .
git commit -m "Initial commit"
git remote add origin <your Github URL here>
git push -u origin master
```

Now you can make sure your Home Assitant configuration is "backed up" to Github. 

### Workflow

This also allows for the workflow where you can make changes on your local workstation and push to github, then pull on the Raspberry Pi. Let's try that now.

On your local workstation

```shell
git clone <your Github URL here> homeassistant
cd homeassistant
code .
```

I'm using Visual Studio code here... if you are not a) you should check it out b) feel free to insert your text editor of choice here.

Once in my code editor I'm jsut going to change the name of my Home Assistant instance from Bargelt Home to Bargelt Home 2, save those changes. 

Back on the command line commit and push your changes:

```shell
git commit -am "Updated the name of the Home Assistant instance"
git push origin master
```

Pull those changes to your Raspberry Pi:

```shell
ssh pi@<Your IP address>
cd homeassistant
git pull
```

Let's restart our container to pickup the change file:

```shell
docker-compose -p ha stop
docker-compose -p ha up -d
```

We could also just `docker restart ha_home-assistant_1`.

### Conclusion

We now have a (very) basic Home Assistant configuration up and running in Docker on our Raspberry Pi. Over the next few blog posts we will explore expanding our system. Now this does not mean that I will or can walk you through configuring HA itself, as we all have different hardware, switches, setups. What I will provide is a how-to for setting up commonly used features like SSL + a custom domain name, MQTT (both internal and external), ZWave, Open ZWave, Plex, Dasher, and AppDaemon to name a few. All running in Docker, using Docker Compose.

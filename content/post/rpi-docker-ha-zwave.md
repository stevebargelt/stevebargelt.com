---
categories: []
date: '2017-11-21T00:00:00Z'
description: Description
featured_image: /assets/rpi-docker-compose-header.png
draft: true
tags: []
title: Adding ZWave to your Home Assistant
type: post
weight: 0
---

## Home Assistant in Docker and ZWave 

If you are not going to use any ZWave devices you can skip this post altogether. If you are planning to use ZWave devices and you are hosting your Home Assistant instance on a Raspberry Pi in Docker, this post is for you!

I assume you have followed the first posts in this series, if not the steps I take may not work for you. Please go visit those posts to catch up if you have not already.
[Link to first posts](https://www.bargelt.com)

[home-assistant.io ZWave Guide](https://home-assistant.io/docs/z-wave/)

### Recognizing the ZWave Controller

Let's SSH into our Raspberry Pi and make a few changes to our docker-compose.yml file and to our HA configuration.

```shell
ssh pi@<your IP address>
cd homeassistant
nano docker-compose.yml
```

At the end of the file add:

```yaml
    devices:
      - "/dev/ttyUSB0:/zwaveusbstick:rwm"
```

CTRL-X, Y, Enter to save your changes

```shell
nano configuration.yaml
```

At the end of the file add:

```yaml
zwave:
  usb_path: /zwaveusbstick
```

**Plug your ZWave controller stick into the Raspberry Pi**

Let's restart our container to pickup the change file:

```shell
docker-compose -p ha stop
docker-compose -p ha up -d
```

We could also just `docker restart ha_home-assistant_1`.

### Update our makefile

Since we restart our Docker container quite often we should add a make command as well

```shell
nano makefile
```

At the end of the file add:

```makefile
restart: stop run
```

Awesome, right? Now hit the web UI in your browser again http://192.168.1.6:8123 and... wait, what?

[IMAGE(https://www.bargelt.com)

The zwave devices are not configured. How do we do that? There are two ways to accomplish this, the first is by using the HA interface. I find this to be confusing. Maybe it's just me? The second way:

### ZWave Configuration with Open ZWave (OZW)

We need to run a utility named Open ZWave in order to configure our ZWave devices. First we must stop Home Assistant:

```shell
docker stop ha_home-assistant_1
```

  OR

  ```shell
  make stop
  ```

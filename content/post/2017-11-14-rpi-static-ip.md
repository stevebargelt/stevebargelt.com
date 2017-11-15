+++
aliases      = []
categories   = ["IoT"]
date         = "2017-11-15T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = false
slug         = ""
tags         = ["raspberry pi", "rpi", "setup", "headless", "ethernet", "static IP"]
title        = "Static IP and Disable WiFi on a Raspberry Pi"
type         = "post"
weight       = 0
+++

### Drive corruption, Containers, and Static IPs. Oh My!

Shortly after publishing [the first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get a headless Raspberry Pi setup, the flash drive in my Home Automation Pi (running Home Assistant) became corrupted. No real problem, as I can easily recreate the setup since I use Docker containers and have my configuration in Github. I popped a new flash drive in the RPi and it would not get the same IP Address from my router. I tried for at least an hour to get my router to provision the same IP to my Pi. I finally gave up and decided to just configure my Pi with it's own static IP.

### Static IP for your Raspberry Pi

SSH into your Pi (using **your** IP address, of course):

```bash
ssh pi@192.168.1.5
```

Now we edit /etc/dhcp.conf...

```bash
sudo nano /etc/dhcpcd.conf
```

Look for the line: "# Example static IP configuration:"

Then uncomment the lines underneath and edit based on your network configuration:

```shell
interface eth0
static ip_address=192.168.1.5/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

CTRL-X, Y, Enter to save your changes.

Reboot your device:

```shell
sudo reboot
```

### Disable Wifi

I know, I know I said that I hardly ever use wired LAN connections any more but for a few things I think it's worthwhile. One place I use a wired connection is my Home Automation server

```bash
sudo nano /etc/modprobe.d/raspi-blacklist.conf
```

add:

```bash
blacklist brcmfmac
blacklist brcmutil
```

CTRL-X, Y, Enter to save your changes.

Reboot your device:

```shell
sudo reboot
```

### Conclusion

Well this was short and sweet. Now we have a static IP for our Raspberry Pi and we've disabled the WiFi (well *I* have - you don't have to disable your WiFi).

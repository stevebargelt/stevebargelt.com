---
categories: []
date: '2017-11-20T00:00:00Z'
description: Description
featured_image: /assets/rpi-docker-compose-header.png
draft: true
tags: []
title: Home Assistant Custom Domain and SSL
type: post
weight: 0
---

## Custome Domain and SSL 


Lets Encrypt:
[Home Assistant Link](https://home-assistant.io/blog/2015/12/13/setup-encryption-using-lets-encrypt/) - won't work on a Pi (x86 vs. ARM)

Thanks to [dude](https://hub.docker.com/r/bcecchinato/letsencrypt-rpi/)

```shell
sudo docker run -it --rm -p 80:80 --name certbot \
                -v "/etc/letsencrypt:/etc/letsencrypt" \
                -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
                bcecchinato/letsencrypt-rpi:latest certonly \
                --standalone --preferred-challenges http-01 \
                --email steve@bargelt.com -d control.bargelt.com
```

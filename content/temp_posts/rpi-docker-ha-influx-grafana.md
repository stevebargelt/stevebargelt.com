+++
aliases      = []
categories   = []
date         = "2017-09-23T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = true
slug         = ""
tags         = []
title        = "HomeAssistant with InfluxDB and Grafana"
type         = "post"
weight       = 0
+++
# Homeassistant Setup : Raspberry Pi and Docker With Grafana and InfluxDB

```shell
mkdir grafana
mkdir influxdb
git clone https://github.com/stevebargelt/home-assistent-config homeassistant
```

if copying config
```shell
cp -r /media/pi/fob/grafana/* grafana
cp -r /media/pi/UBUNTU\ 16_0/influxdb/* influxdb
cp -r /media/pi/fob/secrets.yaml /homeassistant

```

## Configure your Influx DB

docker exec -it ha_influxdb_1 /usr/bin/influx

CREATE DATABASE homeassistant

SHOW DATABASES

use homeassistant

CREATE USER admin WITH PASSWORD 'correcthorsebatterystaple' WITH ALL PRIVILEGES

CREATE USER homeassistant WITH PASSWORD 'steel2000'
GRANT READ ON homeassistant TO "homeassistant"
GRANT WRITE ON homeassistant TO "homeassistant"

exit

<!-- Create alias
alias vault='docker exec -it vault-dev vault "$@"' -->


## Configure Grafana

## Create Dashboards


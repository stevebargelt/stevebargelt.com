+++
aliases      = []
categories   = []
date         = "2017-10-25T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = true
slug         = ""
tags         = []
title        = "Creating a Build Radiator with an RPi"
type         = "post"
weight       = 0
+++
See [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose as we will be building on that foundation to get our build radiator set up.

## Why a build radiator?
Pretty simple, really; visibility. The more visible you make problems, the more attention they get, the quicker they get solved. In 2017 most of us have pretty good mean of filtering out email, Slack, Twitter, and the like. If a red light flashes and a buzzer goes off in the physical world, it is hard to ignore. In addition when you have multiple teams contributing to one codebase it helps to remind teams to stop and swarm if any build breaks. 


## Found a solution out there
Link to Python solution...

```shell
ssh pi@192.168.1.6
mkdir jenkins-py
git clone https://github.com/BramDriesen/rpi-jenkins-tower-light.git
cp rpi-jenkins-tower-light/default-config.py config.py
nano config.py
```

```shell
jenkinsurl = "https://abs.harebrained-apps.com"
username = "jenkinsuser"
password = "correcthorsebatterystaple"
jobs = ['shoppingcart-aspdotnetcore', 'testJob']
gpios = {
    'red': 18,
    'buzzer': 23,
    'yellow': 24,
    'green': 27,
}
```

So now we have **our** configuration settings in the config file. 

## Dockerize all the things

We don't want to just run this code... we want to run it in a Docker container. Why? Python 3.6. etc. 

```shell
nano Dockerfile
```

```docker
FROM resin/raspberrypi3-python:3.6

WORKDIR /usr/src/jenkins-py

COPY  rpi-jenkins-tower-light/jenkinslight.py ./
COPY  rpi-jenkins-tower-light/requirements.txt ./

RUN pip install -r requirements.txt

# Define default command
CMD ["python", "-u", "./jenkinslight.py"]
```

Should note here that the -u un-buffers the Python output. For the longest time I thought there was a problem with Python writing to the stdout (via print, etc.) because I never saw anything in `docker logs jenkins-py` - well I guess it was buffereing.

Pi 2 and 3 (ARM x)
https://hub.docker.com/r/resin/raspberrypi3-python/

Pi Zero and Pi 1 (ARM x)
https://hub.docker.com/r/resin/raspberry-pi-python/


```shell
docker build -t jenkins-py .

docker run -d --name jenkins-py -v $(pwd)/config.py:/usr/src/jenkins-py/config.py --cap-add SYS_RAWIO --device /dev/mem --privileged jenkins-py

docker run -d --restart=always --name jenkins-py -v $(pwd)/config.py:/usr/src/jenkins-py/config.py --cap-add SYS_RAWIO --device /dev/mem --privileged jenkins-py

```

To get into the docker container

```shell
docker exec -it jenkins-py /bin/sh
```

To see the logs:

```shell
docker logs jenkins-py
```

docker-compose.yml

docker-compose.yml is:

```
version: '3'
services:
  jenkins-watcher:
    image: jenkins-py
    privileged: true
    restart: always
    volumes:
        - /home/pi/jenkins-py/config.py:/src/config.py
    cap_add:
      - SYS_RAWIO 
    devices:
      - "/dev/mem:/dev/mem"
```


### Other Links

https://github.com/DiUS/build-lights

https://blog.hypriot.com/post/docker-sensor-fu-on-a-raspberry-pi/

https://blog.hypriot.com/post/lets-get-physical/

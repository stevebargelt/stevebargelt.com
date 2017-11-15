+++
aliases      = []
categories   = []
date         = "2017-11-16T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = false
slug         = ""
tags         = []
title        = "Creating a Build Radiator with an RPi"
type         = "post"
weight       = 0
+++

See [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose as we will be building on that foundation to get our build radiator set up.

## Why a build radiator?

Pretty simple, really; visibility. The more obvious you make your problems, the harder you make them to ignore - the more attention they get and the quicker they get solved. In 2017 most of us have pretty good means of filtering out email, Slack, Twitter, and the like. If a red light flashes and a buzzer goes off in the physical world, it is hard to ignore. In addition when you have multiple teams contributing to one codebase it helps to remind teams to stop and swarm if any build breaks. I will caution that if your team(s) ignore the red light then you have deeper issues that you need to address.

## Pre-baked Solution

 To get up and running quickly I did a search for solutions out there and I ran across this Link to [Python solution on Github](https://github.com/BramDriesen/rpi-jenkins-tower-light.git). One advantage is that I knew I could get this up and running very quickly. The disadvantage is that it only works with Jenkins and I currently have projects using Travis CI, Jenkins, Drone.io, and Netlify. I'm a huge fan of getting something working, anything. Then you can iterate and refine the product, adding features. I also looked at the code and setup and decided it would be easy to run everything in a Docker container.

## Hardware setup V1

Eventually we want a setup just like Bram Driesen shows in his excellent article but for starters I'm going to use a simple setup to ensure that the code and logic is working. This is my minimum viable product or MVP. In a future post I'll walk through upgrading to the [Tower Light](https://www.adafruit.com/product/2993) and eventually to Neopixels (have not decided on the final product yet.)

Here is a basic diagram of the V1 (MVP) setup:
{{< blog-img src="/assets/rpi-build-radiator-001-fritzing" alt="Fritzing Diagram of the V1 hardware" >}}

## Clone the solution

I did have to make a couple modifications to the source code so I forked the original repository to [https://github.com/stevebargelt/rpi-jenkins-tower-light](https://github.com/stevebargelt/rpi-jenkins-tower-light).

First, we will SSH into our Raspberry Pi. If your Pi isn't setup for this see [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}). (and obviously use *your* IP address!)

```shell
ssh pi@192.168.1.6
```

Then clone the repository:

```shell
mkdir jenkins-py
cd jenkins-py
git clone https://github.com/stevebargelt/rpi-jenkins-tower-light
```

Next we need to copy and edit the config file:

```shell
cp rpi-jenkins-tower-light/default-config.py config.py
nano config.py
```

Edit the config to model your Jenkins settings and your LED GPIO pins. In Jenkins, I'd suggest creating a new user just for this purpose so you can limit the access rights and disable the account should the config file be compromised.

```shell
jenkinsurl = "https://abs.harebrained-apps.com"
username = "buildlightuser"
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

We don't want to just run this code... we want to run it in a Docker container. Why? Python 2.7 (the default version on our Raspberry Pi Raspbian image) is the past and I'd rather not install and update Python versions on my Raspberry Pi. For all the reasons I listed in the first part of the blog series we want to containerize as much as we possibly can. The main benefit is in my opinion is the repeatability of the process. If we run this in a Docker container, we can easily remove it from the device and re-purpose it. We can also easily update to a new base image (a new version of Python). It keep our base OS image as clean as possible.

Use your shell editor of choice to create a new Dockerfile:

```shell
nano Dockerfile
```

The Docekrfile contents will look like this:

```docker
FROM resin/raspberrypi3-python:3.6

WORKDIR /usr/src/jenkins-py

COPY  rpi-jenkins-tower-light/jenkinslight.py ./
COPY  rpi-jenkins-tower-light/requirements.txt ./

RUN pip install -r requirements.txt

# Define default command
CMD ["python", "-u", "./jenkinslight.py"]
```

I will note here that the -u un-buffers the Python output. For the longest time I thought there was a problem with Python writing to the stdout (via print, etc.) because I never saw anything in `docker logs jenkins-py` - apparently it was buffering.

If you are using a Raspberry Pi 2 or 3 then the above Dockerfile will work just fine. If you are using a Raspberry Pi 1 or Raspberry Pi Zero or Zero W then you will need to use `FROM resin/raspberry-pi-python:3.6` - See the following for more information: [Pi Zero and Pi 1](https://hub.docker.com/r/resin/raspberry-pi-python/).

Next up, build the Docker image:

```shell
docker build -t jenkins-py .
```

Then we run that image:

```shell
docker run -d --name jenkins-py -v $(pwd)/config.py:/usr/src/jenkins-py/config.py --cap-add SYS_RAWIO --device /dev/mem --privileged jenkins-py
```

If all is well with our setup **and** our Jenkins jobs, the green LED should illuminate.

## Troubleshooting

If all is not well with our setup then here are a few troubleshooting tips.

To get into the docker container to poke around:

```shell
docker exec -it jenkins-py /bin/sh
```

To see the logs:

```shell
docker logs jenkins-py
```

## Quick Example

If I go into my Jenkins web front-end and sabotage my test job build by adding nonsense to the pipeline code
{{< blog-img src="/assets/rpi-build-radiator-001-jenkins-pipe" alt="" >}}

the job will fail
{{< blog-img src="/assets/rpi-build-radiator-001-jenkins-fail" alt="" >}}

We should see a red light come on...

## Auto Starting the Container

If everything looks legit then we can tell Docker to always restart this container.

```shell
docker stop jenkins-py
docker rm jenkins-py
docker run -d --restart=always --name jenkins-py -v $(pwd)/config.py:/usr/src/jenkins-py/config.py --cap-add SYS_RAWIO --device /dev/mem --privileged jenkins-py
```

## Docker Compose
If you'd prefer to use Docker compose here is the docker-compose.yml file:

```docker
version: '3'
services:
  jenkins-py:
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

```shell
docker-compose -p buildwatcher up -d
```

### Conclusion

We now have a Minimum Viable Product of a build status radiator that you can show your boss and your co-workers in order to get them on-board to move to the next step... in the next installment we will add a real Tower Light on the hardware side of the fence.


<!-- ### Other Links

https://github.com/DiUS/build-lights

https://blog.hypriot.com/post/docker-sensor-fu-on-a-raspberry-pi/

https://blog.hypriot.com/post/lets-get-physical/

Hardware V3 https://blog.adafruit.com/2016/05/31/tower-light-upgraded-with-particle-photon-neopixels-and-ifttt-iot-iotuesday/

Java
https://github.com/Capoot/build-status-traffic-light

Siren of Shame
https://github.com/AutomatedArchitecture/SirenOfShame -->

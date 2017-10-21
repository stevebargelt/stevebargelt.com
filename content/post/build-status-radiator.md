+++
aliases      = []
categories   = []
date         = "2017-10-11T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-header.png"
draft        = true
slug         = ""
tags         = []
title        = "Creating a Build Radiator with an RPi"
type         = "post"
weight       = 0
+++
See [The first post in this series]({{< ref "2017-10-09-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose as we will be building on that foundation to get our build radiator set up.

## Why a build radiator?

## Custom code

```docker
FROM resin/raspberrypi3-python:3.6

VOLUME /conf

# Copy appdaemon into image
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . .

# Install
RUN pip3 install .

# Start script
RUN chmod +x /usr/src/app/dockerStart.sh
CMD [ "./dockerStart.sh" ]
```
https://hub.docker.com/r/resin/raspberrypi3-python/

Pi Zero and Pi 1
https://hub.docker.com/r/resin/raspberry-pi-python/


```shell
mkdir jenkins-py
cd jenkins-py
git clone https://github.com/BramDriesen/rpi-jenkins-tower-light.git
cp rpi-jenkins-tower-light/default-config.py config.py
nano config.py
```

```shell
# Default configuration file for Jenkins
# Copy this file and name it config.py
jenkinsurl = "https://abs.harebrained-apps.com"
username = "jenkinsuser"
password = "correcthorsebatterystaple"
jobs = ['shoppingcart-aspdotnetcore']
gpios = {
    'red': 18,
    'buzzer': 23,
    'yellow': 24,
    'green': 27,
}
```

To your settings for Jenkins

```
nano Dockerfile
```
Dockerfile is:

```
FROM resin/rpi-raspbian:jessie
MAINTAINER Steve Bargelt <steve@bargelt.com>

# Install dependencies
RUN apt-get update && apt-get install -y \
    python \
    python-dev \
    python-pip \
    python-virtualenv \
    python-jenkinsapi \
    python-rpi.gpio \
    git \
    --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

COPY  rpi-jenkins-tower-light /src

# Define default command
CMD ["python", "/src/jenkinslight.py"]

```

```
docker build -t jenkins-py .

docker run -d --restart=always --name jenkins-py -v $(pwd)/config.py:/src/config.py --cap-add SYS_RAWIO --device /dev/mem jenkins-py

```

docker-compose.yml

docker-compose.yml is:

```
version: '3'
services:
  jenkins-watcher:
    image: jenkins-py
    restart: always
    volumes:
        - /home/pi/jenkins-py/config.py:/src/config.py
    cap_add:
      - SYS_RAWIO 
    devices:
      - "/dev/mem:/dev/mem"
```

# Wrting the code in a 'real' language

On your local workstation (Mac in my case)

```
go get -d -u gobot.io/x/gobot/...
```

Create your main.go 

Compile 

```
GOARM=7 GOARCH=arm GOOS=linux go build main.go
```

```
scp main pi@rpi-watcher:/home/pi/
ssh -t pi@rpi-watcher "./main"
```


# Gulp and Browserify 

```
npm install --save gulp gulp-connect gulp-open

npm install --save bootstrap jquery gulp-concat

npm install --save gulp-eslint

** npm install --save react react-dom react-router flux

npm install --save prop-types

** npm install --save-dev babel-preset-es2015 babel-preset-react

npm install --save-dev babelify


```


gulp.task('concat', ['copy-react', 'eslint'], function() {
  return gulp.src(jsFiles.vendor.concat(jsFiles.source))
    .pipe(sourcemaps.init())
    .pipe(babel({
      only: [
        'assets/js/src/components',
      ],
      compact: false
    }))
    .pipe(concat('app.js'))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('assets/js'));
});

# React Router v4

npm install --save gulp gulp-connect gulp-open

npm install --save bootstrap jquery gulp-concat

npm install --save gulp-eslint

npm install --save react react-dom

npm install --save prop-types

npm install --save-dev babelify

npm install --save react-router-dom


https://codesandbox.io/s/vVoQVk78?referrer=https%3A%2F%2Fmedium.com%2Fmedia%2Fcb2f4eec602746212e3d562340fb8898%3FpostId%3D7f23ff27adf

https://medium.com/@pshrmn/a-simple-react-router-v4-tutorial-7f23ff27adf



https://github.com/coryhouse/react-slingshot
^^ not to Router 4.x yet tho - PR out there to get to 4.x

# React and Redux 

```

```
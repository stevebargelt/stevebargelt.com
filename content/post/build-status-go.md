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

# Writing the code in a 'real' language

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
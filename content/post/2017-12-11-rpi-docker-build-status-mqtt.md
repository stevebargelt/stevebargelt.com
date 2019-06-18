---
categories: []
date: '2017-12-11T00:00:00Z'
description: This is a description
featured_image: /assets/rpi-docker-compose-homeassistant.png
draft: true
tags: []
title: MQTT Build Status Radiator System
type: post
weight: 0
---

## Introduction

See [The first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose as we will be building on that foundation to get Home Assistant set up.

Next Up HomeAssistant Post w/ MQTT

Light bulb went off... we are doing this whole build status thing wrong. Pub/sub seems correct! YES!


## Pure MQTT Clients

{{< blog-img src="/assets/rpi-build-radiator-002-firstpass" alt="Arch diagram of my first thought." >}}

MQTT is the answer, our sources should publish messages to a MQTT broker (or to a webhook => MQTT translator => MQTT broker). I actually did a bunch of work writing a Webhook to MQTT broker translation layer in Azure Functions. Then it hit me... what happens when Project A fails (send mqtt on channel build/projecta/ of fail) we turn the Green Light off and turn Red light on. Simple! I win the Interwebs! Then Project B succeeds (send build/projectb of success) so we turn off the Red light and turn on the Green light. Wait... Project A is still failing. I started to write Arduino code to track projects and hold state. This is exactly what I was trying to avoid. Back to the drawing board. 

## A Controller Service (to MQTT)

Next idea that popped into my head was to have a controller "service" in the could that all of the build systems could call via webhook (as an added bonus, the controller can call the service (Jenkins I'm looking at you) if the service does not have native Webhooks) the service would potentially have persistent storage (depicted in the diagram as a database, but it could be as simple as JSON in text files on the fiel system). 

{{< blog-img src="/assets/rpi-build-radiator-002-sendondpass" alt="Arch diagram of my second thought." >}}



## If you have your own MQTT broker already running

### Add users to MQTT

```shell
docker exec -it ha_mqtt-ext_1 /bin/sh
```

if this is your first user you need to use the -c to create the file..

```shell
mosquitto_passwd -c /etc/mosquitto/pwfile jenkinsuser
```

If you are already running MQTT with a user then **DO NOT** use the -c flag 0 that will kill the old file.  Just omit the -c to add users to the file...

```shell
mosquitto_passwd /etc/mosquitto/pwfile jenkinsuser
```

Copy the contents of the pwfile

```shell
cat pwfile
```


exit


### Using Azure IoT as your MQTT Broker

https://webhookmqtt.azurewebsites.net/api/GithubWebhookMQTT?clientId=default

ha7Mc0NfIpqypsJaFHOzDdaSU84xsPV/jOXeh2p10Ti0fDCOpv5OMw==


https://webhookmqtt.scm.azurewebsites.net


Travis webhooks are encoded:
  application/x-www-form-urlencoded

https://chriskirby.net/blog/fun-with-slack-commands-using-azure-api-and-function-apps


function.json
```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "github",
      "name": "req"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ],
  "disabled": false
}
```
index.js
```javascript
// Please visit http://go.microsoft.com/fwlink/?LinkID=761099&clcid=0x409 for more information on settting up Github Webhooks
module.exports = function (context, data) {
    context.log('GitHub Webhook triggered!', data.comment.body);
    context.res = { body: 'New GitHub comment: ' + data.comment.body };
    context.done();
};
```


### Links

Jenkins MQTT Plugin
[https://wiki.jenkins.io/display/JENKINS/MQTT+Notification+Plugin](https://wiki.jenkins.io/display/JENKINS/MQTT+Notification+Plugin
)
Github Hooks
[http://mqtt.org/2012/05/mqtt-service-hook-added-to-github](http://mqtt.org/2012/05/mqtt-service-hook-added-to-github)


Azure IoT 
[https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support)



https://tech.scargill.net/mosquitto-and-web-sockets/

http://blog.ithasu.org/2016/05/setting-up-mosquitto-on-raspbian-jessie/


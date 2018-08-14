+++
aliases      = []
categories   = []
date         = "2017-12-07T00:00:00Z"
description  = ""
featured_image = "/assets/rpi-docker-compose-jenk-trav-light.png"
draft        = false
slug         = ""
tags         = []
title        = "Adding the Tower Light to our Build Radiator"
type         = "post"
weight       = 0
+++

See [the first post in this series]({{< ref "2017-10-20-rpi-01-docker.md" >}}) to get your Raspberry Pi setup with Docker and Docker Compose, then [the second post in this series]({{< ref "2017-11-16-rpi-docker-build-status-radiator.md" >}}) to get the minimum viable product setp. In this post we will replace our single LEDs with a [Tower Light](https://www.adafruit.com/product/2993) from adafruit.

## The Tower

As I said when w started building a build radiator - this is all about visibility. The tower light we are adding to our system helps us achieve that goal partly because it is very bright. It can achieve this because it runs on 12v. Our Raspberry Pi GPIOs are only 3.3v. We can no longer power the light directly from the Pi. We will need an external 12v power supply. So how do we use our 3.3v output to switch 12v? We can use transistors as switches. 

Note:

>I assume you’re on a computer similar to mine (a MacBook Pro running MacOS). It's certainly not a requirement. You could be using Linux or Windows, though some of the client-side tooling changes (PuTTY for Windows SSH, as an example). I'm confident that if you are interested in the fairly advanced concepts I'm presenting here, you will figure it out and translate to your platform of choice.


## Parts List

* 1x [Adafruit 12V LED Tower light](https://www.adafruit.com/product/2993)
* 4x [NPN Transistros (IRLB8721)](http://amzn.to/2jMiyDY)
* 4x [4300 Ohm resistor](http://amzn.to/2Ao38zE)
* 1x [DC Barrel power jack](http://amzn.to/2igjRdR)
* 1x [12V To USB converter](http://amzn.to/2AuYKwy)
* 1x [DC 12V Power adapter](http://amzn.to/2jOBwd1)
* 1x [Breadboard](http://amzn.to/2iiTcgI)
* [Heat-shrink tubing](http://amzn.to/2AubSSP)
* [22AWG Solid copper wire](http://amzn.to/2ApQPCP)


## Hardware setup V1

The goal of this post is to get the Tower Light working. Here is a basic diagram of the V2 setup:
{{< blog-img src="/assets/rpi-build-radiator-v2" alt="Fritzing diagram of the V2 hardware" >}}

The (ugly) right side of the diagram, the LEDs and buzzer are just a representation of the tower light. Basically there is one power lead (the brown wire) and the rest are ground wires, three for the lights (correspond to the light color) and an orange wire for the buzzer.

Pictures of the V2 Product:
{{< blog-img src="/assets/rpi-build-tower-1" alt="" >}}
{{< blog-img src="/assets/rpi-build-tower-2" alt="" >}}
{{< blog-img src="/assets/rpi-build-tower-3" alt="" >}}
{{< blog-img src="/assets/rpi-build-tower-4" alt="" >}}

The important change here is that we are using an external 12v power supply to power the Tower Light. We are also using it to power our Raspberry Pi through the use of a DC-DC Buck Converter - the one I am using coverts the 12v power to USB (5v) to be fed back into the Raspberry Pi. You do not need to power the Pi via the micro-usb port, you can, instead, power it via the 5v and GND pins, but I had this converter lying around. Adding the extremely short USB cable does add to the bulk of the total build so I may order a difference buck converter before I transfer this to a project box.

Once you have everything hooked up you can restart your Raspberry Pi and if you hooked everything up correctly and your build are passing the green light should come on!

## Conclusion

We now have v2 of our build status radiator that you can show off and show progress with. In the next post I will transfer this to a [perma-proto board](http://amzn.to/2BKpGsL) and transfer the electronics to project box.
---
layout: post
title:  "Making an indoor self-watering greenhouse"
author: Ulf Lilleengen
categories: technical iot embedded
---

This post should've been written one year ago, when I actually did this project. With another holiday project coming up, it feels necessary to at least post a summary of last years project.

In-door gardening seems to become popular, and rather than spending a lot of money on a nice-looking product like [auk](https://www.auk.eco), I thought I would instead spend twice as much on my own beutiful rig:

![concept](/images/greenhouse/concept.png)

The goal is to monitor some herbs, measuring their soil moisture, and then water the plants with the goal of getting a good harvest. Of course, this can all be made much simpler and mechanical, but where is the fun in that?

The target architecture is as follows:

![architecture](/images/greenhouse/architecture.png)

A microcontroller periodically measures moisture and reports a relative humidity to the cloud. In the cloud, there is an controller running that consumes these events and decides if the plant is too dry or not. If plan is too dry, the controller will send a message back to the microcontroller instructing it to water the plant. A console shows the current and historical state of the plants.

## Hardware

* ESP8266
* Pump + Relay
* Soil moisture sensor
* Artifical light
* Timer for artificial light

As a microcontroller I chose an ESP8266 using a NodeMCU-based board. This microcontroller has a WiFi chip builtin, which makes it easy to connect it to my home network and talk to the cloud serviec.

For watering the plants, I used a [submersive pump](https://www.amazon.com/MOUNTAIN_ARK-Submersible-Amphibious-Hydroponic-Fountains/dp/B010LY7P3Y) mounted at the bottom of a plastic box. To control the pump on/off, I used a relay connected to the microntroller. 

To measure the soil, I initially started off with a sensor from [sparkfun](https://www.sparkfun.com/products/13637), but it was not able to tell the difference from soaking wet to a little moist. I replaced that later with a capacity sensor from [dfrobot](https://wiki.dfrobot.com/Capacitive_Soil_Moisture_Sensor_SKU_SEN0193) that gave more accurate readings.

Finally, to ensure stable light conditions for the plants, I purchased an artifical light lamp and a timer that turns on and off at specific hours during the day.

## Software

### Microcontroller

The microcontroller is implemented using [Arduino + PlatformIO](https://github.com/lulf/dingser/tree/master/wifi-node/esp8266/greenhouse), which works OK for hobby projects.

### Messaging Layer 

This is only needed to push data to/from cloud/devices. You can host this yourselv using the open source projects like [Eclipse Hono](https://www.eclipse.org/hono/) and [EnMasse](https://enmasse.io), or use a commerical services as [Bosch IoT Hub](https://developer.bosch-iot-suite.com/service/hub/).

### Greenhouse Backend

The cloud side of the greenhouse contains several components to handle event data and displaying graphs:

* [Console API](https://github.com/lulf/dings-api) - A GraphQL API for the console to pull event data.
* [Console](https://github.com/lulf/dings-console) - A single page application written in Svelte that uses the GraphQL API to retrieve data.
* [Controller](https://github.com/lulf/greenhouse-controller) - A Go-based server that subscribes to event data and sends commands back to an Eclipse Hono API to water the plants.
* [Hono Sink](https://github.com/lulf/hono-event-sink) - A Go-based server that pulls data from an AMQP messaging service and stores it in an event store.
* [AMQP Event Store](https://github.com/lulf/slim) - A Go-based event store where all event data is stored.

The entire backend is written to run in [Kubernetes](https://kubernetes.io).

## Result

All in all, I'm pretty happy with the result, even though I suck at carpenting:

![Mount](/images/greenhouse/frame_and_microcontroller.jpg)

The whole thing in action:

![Complete](/images/greenhouse/full.jpg)


## Summary

All in all, this was a really fun project, and the basil plants did enjoy the extra care (and I was able to make lots of pesto!). Unfortunately, during our move to a new home, the entire thing got dismantled and I haven't brought it up again. I am thinking about making a larger v2 of this now that we have more space.

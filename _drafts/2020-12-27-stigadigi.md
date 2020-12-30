---
layout: post
title: On digitizing an ice hockey table game
author: Ulf Lilleengen
categories: technical embedded nrf microbit
---

[Stiga Table Hockey](https://www.stigasports.com/eu/leisure-play/table-games/table-hockey) is a classical table game here in Norway, and familiar many. One of my side projects while studying was to implement an automated goal detection mechanism and log results online. I did manage to scrap together a [php](https://github.com/lulf/stigadigi/tree/ecdd4e0f8953c1d14f7a9ecd41f7055be03fa4a3) application to keep track of scores, and some buttons to register goals, but I did not manage to get the goal detection implemented.

# Rethinking the design

Due to a renewed interest in Rust and embedded on my part, I decided to re-implement this project during the corona christmas holidays of 2020.

The original plan was to use a [BBC micro:bit](https://microbit.org) to perform these tasks:

* Read sensor data when puck is inside the goal area
* Handle button events to start and stop games
* Display game state on the LEDs
* Forward game data to a web service

# Exploring sensors

The most important pieces of this work was to find a reliable way to detect goal. The initial approach I took was to drill a few holes in the goal and place a photoresistor and laser emitter on each side of the goal.

This solution proved difficult to mount and it required very accurate positioning of the laser and photoresistor. 

The second approach I tried was to use reed switches (magnetic field detection) and modify the puck with a small magnet. After trying a few differently sized magnets, I found one that would get detected reliably, while not getting stuck on the hockey stick of the players (which is made of metal!).

# Displaying the score

My initial plan was to install a couple of 7-segment displays to show the score. However, the multiplexing these displays required more circuitry, so I postponed that to future work.


# Hardware

The final build contains the following hardware:

* BBC micro:bit v2
* 2 reed switch sensors
* 1 kitronik micro:bit edge connector
* 2x magnets


TODO: Figure

# Software

The controller software is written in Rust with great help from the nRF HAL crates for working with GPIO and PWM.

# Future work

Even though the reed switches are detecting most goals, I still think there is some room for improvement, so I will mount another couple of switches.

Connecting the controller to the cloud is on the road map, and I have an early implementation of the API that integrates with PostgreSQL. The missing piece is to have the controller communicate with internet, and I'm exploring multiple options, the likely approach being to use an ESP8266 for Wi-Fi connectivity.

All in all, its been a fun project, and I look forward to improve it further.

---
layout: post
title: On digitizing an ice hockey table game
author: Ulf Lilleengen
categories: technical embedded nrf microbit
---

[Stiga Table Hockey](https://www.stigasports.com/eu/leisure-play/table-games/table-hockey) is a classical table game here in Norway. One of my side projects while studying was to implement an automated goal detection mechanism and log results online. I did manage to scrap together a [php(!)](https://github.com/lulf/stigadigi/tree/ecdd4e0f8953c1d14f7a9ecd41f7055be03fa4a3) application to keep track of scores, and some buttons to register goals, but I did not manage to get the goal detection implemented. As much fun as it can be to get a 10 year old web application to run again, I figured it was time for a rewrite.

Due to a renewed interest in Rust and embedded on my part, I decided to re-implement this project during the corona christmas holidays of 2020. All of the software is available on [github](https://github.com/lulf/stigadigi).

# Rethinking the design

The original plan was to support the following in the new design:

* Detect when puck is inside the goal area
* Handle button events to start and stop games
* Display game state for the players
* Forward live game data to a web service 

# Goal detection

The most important pieces of this work was to find a reliable way to detect goal. The initial approach I took was to drill a few holes in the goal and place a photoresistor and laser emitter on each side of the goal.

Although this mechanism detected a goal with high accuracy, it proved difficult to mount the sensors as it required very accurate positioning of the laser and photoresistor. 

The second approach I tried was to use reed switches (magnetic field detection), drill a hole in the puck and embed a small magnet. After trying a few differently sized magnets, I found one that would get detected reliably, while not getting stuck on the hockey stick of the players (which is made of metal!).

![/images/IMG_5633.JPG](Puck)

Mounting the reed switch in the goal area was also quite simple:

![/images/IMG_5635.JPG](Reed switch)

# Start and stop events

The micro:bit cotnains two buttons. The left button will start/stop the game, and the right button will undo the last operation, in the event of a wrongly detected goal.

# Displaying the score

My initial plan was to install a couple of 7-segment displays to show the score. However, the multiplexing these displays required more circuitry, so I postponed that to future work. Luckily, the micro:bit has a 5x5 LED grid, and I decided to use that to display the score.

# Forwarding live game data

I started writing a REST API that the controller would talk to to store the game state. The API will eventually also be able to update the player rating depending on which players played against each other. Due to lack of time I decided to postpone this work, but the initial work is stored in the repository.

# Hardware

The final build contains the following hardware:

* [BBC micro:bit v2](https://microbit.org)
* [2 reed switch sensors](https://en.wikipedia.org/wiki/Reed_switch)
* [1 kitronik micro:bit edge connector](https://kitronik.co.uk/products/5601b-edge-connector-breakout-board-for-bbc-microbit-pre-built)
* [2x magnets](https://panduro.com/nb-no/products/skap-dekorer/dekormaterialer/magneter/supermagneter-5-mm-8-stk-731506?gclid=Cj0KCQiA0MD_BRCTARIsADXoopZKlUlj-e5vbmtTgQk6C4E9O56M1JdxdqHY9Q0D1D-SrRs-FKHTrV8aAm9KEALw_wcB#fo_c=3247&fo_k=b9dcf4a03db370cefee944a3aadadb1a&fo_s=gplano)

# Software

The [controller software](https://github.com/lulf/stigadigi) is written in Rust with great help from the nRF HAL crates for working with peripherals. The controller maintains a game state that is initialized when a game is started. The controller maintains the entire game state and all events throughout a game until one of the players reach the "winning score". 

# Future work

![/images/Fil_003.jpeg](Finished!)

Even though the reed switches are detecting most goals, I still think there are some blind spots after testing, so I will mount another couple of switches.

I'd also like to utilize the PWM to generate some sound effects.

Connecting the controller to the cloud is on the road map, and I have an early implementation of the API that integrates with PostgreSQL. The missing piece is to have the controller communicate with internet, and I'm exploring multiple options, the likely approach being to use an ESP8266 for Wi-Fi connectivity.

All in all, its been a fun project, and I look forward to improve it further.

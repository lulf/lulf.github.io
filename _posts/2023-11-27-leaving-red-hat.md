---
layout: post
title: Leaving Red Hat
author: Ulf Lilleengen
categories: non-technical work open source
---

Today (November 27th) is my last day at Red Hat. I joined Red Hat in May 2016, so it's over 7 years, some of the most eventful years of my life so far. I moved from Trondheim to Hamar, became father for the second time, and found a new home where I'll likely be staying for a long time.

It's been almost 3 years since I posted on this blog last time, so this is a good time to reflect on my time there. Hopefully it will start a new trend with more regular updates.

## Nostalgia

The first thing that I remember after joining was that Red Hat felt like the totally right place for me. Open source mindset throughout the organization and a place where good ideas could come from anyone and anywhere. I got to work on a new project that we eventually named [EnMasse](enmasseproject.github.io), which I've written about previously on this blog.

At the yearly face to face gatherings I got to know others from the team, and made many new friends. As the years passed, we grew the team and was able to meet customers using the associated products. I traveled to conferences to speak about the project and learned a lot about building communities. I think the most valuable lesson from this period was how to work with others remotely and that even if you are remote, a periodic refresh with face-to-face meeting is still needed to better understand each other.

Then one day in December 2020 I got the opportunity to join another team at Red Hat that was using Rust to build an Internet of Things ecosystem.

## Re-discovering embedded

My previous blog post was written just after I decided to join my new team, working on the [drogue.io](https://drogue.io) project I have always been interested in embedded, and Rust, but hadn't yet tried to combine those two. From past experience, embedded C was painful and I got a feeling of being unproductive compared to using higher level languages. 

With Rust, all of that changed for me. The programming language itself reminded me about the good parts of Haskell, and the toolchain turned out to be very close to regular application development. Finally I re-discovered the world of embedded, and during the years on the Drogue IoT team, I learned more than I ever have. Thanks to my team mates and fantastic manager, I got to work on these things:

* A embedded [TLS 1.3](https://github.com/drogue-iot/embedded-tls) implementation
* An [Actor Framework](https://github.com/drogue-iot/ector)
* An embedded [HTTP client](https://github.com/drogue-iot/reqwless)
* A [Bluetooth Mesh](https://github.com/drogue-iot/btmesh) stack
* A [LoRaWAN](https://github.com/ivajloip/rust-lorawan) implementation
* [Embassy](https://embassy.dev/) - framework for embedded applications
* A [PCB](https://blog.drogue.io/pcb-part-1/)
* A [bootloader](https://blog.drogue.io/firmware-updates-part-1/)

All of the above tied into an IoT ecosystem for both bare metal and Linux devices that integrated with Red Hat technologies on the backend side. Most blog posts during that can be found on the [Drogue IoT Blog](https://blog.drogue.io).

## Moving on

Eventually our team had to move on to different work, but I had already been convinced I wanted to pursue embedded further. Working in upstream communities as one does at Red Hat allows you to connect with lots of other people, and I was lucky to find my next place this way. 

A big thank you to all my great coworkers for the past 7 and a half years, I hope we will bump into each other at some conference.

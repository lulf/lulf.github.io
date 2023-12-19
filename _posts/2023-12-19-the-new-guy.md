---
layout: post
title: The new guy
author: Ulf Lilleengen
categories: non-technical work
---

On December 4th, I started working for [Akiles](https://akiles.app), a [PropTech](https://en.wikipedia.org/wiki/Property_technology) company selling products such as door locks integrated with a cloud service for access control. In the past three years, I've spent a lot of time creating generic IoT infrastrucure both on the embedded and backend side, and I'm very excited to experience IoT in a real world use case.

Another great thing is that Akiles has been using Rust for their firmware for a while, and the [Embassy](https://embassy.dev) project was started by one of the founders. Having already worked together upstream on the same code base was also a good way to understand a little of what I was getting into.

<div align="center">
<figure>
    <img src="/images/flowers.jpg"
         alt="A photo showing flowers I received from my wife on the first day in my new job"
         width="50%">
    <center><figcaption>Flowers from my wife on my first day.</figcaption></center>
</figure>
</div>

# Size matters

Having only worked at companies with a lot of employees, I was curious what it would be like at a small company.

The first thing I noticed is that there is no information overload. At big companies, there are always some kind of announcement in all places, which requires dicipline to avoid distracting onself. 

The second thing I realized is that the entire company is in the same chat rooms and the minimal communication overhead as a result. Also there are no forms you have to fill out to perform your work (such as contributing to an open source project, looking at you Yahoo! 2010-edition).

# A fresh start

During the first week I've spent a lot of time learning about the products and the software. 

One thing I've kept in mind is to give myself the time and space to listen and learn. Luckily there were some good starter tasks in the backlog that got me going, where I could use to learn different aspects of the system, and I've already been able to add some features to the firmware (Rust) and server backend (Go) this way. 

<div align="center">
<figure>
    <img src="/images/akiles.jpg"
         alt="A photo showing the different Akiles products like cylinder lock, interior doorlock and some example access cards"
         width="50%">
    <center><figcaption>A sample of products that I will be working on.</figcaption></center>
</figure>
</div>

# Open source

So what about open source? The reality is that a lot of the technology stack is not open source, which I think is unfortunate, but it's understandable that this is has not been
a priority, and it's anyway not my focus right now. However, there are many opportunities for contributing to projects like Embassy and libraries that are part of the firmware. For instance I recently did some work on the [sequential-storage](https://crates.io/crates/sequential-storage) queue mechanism as a result of requirements in our stack.

# Remote

I've been a remote worker or almost 8 years now, and one thing I've realized is that even though working together remotely from day to day works well, over time the in-person meetings are important for maintaining a 'social connection', 'mutual understanding' and feeling like one team. So next year I'm looking forward to visit my new colleagues in Barcelona and say hello.

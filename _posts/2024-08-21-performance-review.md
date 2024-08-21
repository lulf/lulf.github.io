---
layout: post
title: Performance review
author: Ulf Lilleengen
categories: non-technical work
---

I'm celebrating 9 months at [Akiles](https://akiles.app), and wanted to share an update on what I've been up to and how it's going, sort of a self-evaluation, or performance review as it's called at "BigCorp".

# Life at a "startup"

When do you stop calling a startup for a startup? I don't know, but working at a small company is certainly different from what I was used to. I think what I've enjoyed most is the ability to focus more on my work with less disruptions, even if there is occasionally something that needs my attention.

One downside of being remote in a small team, is that it is harder to connect with co-workers and a feeling of belonging in a company. However, I think this is something we will have to learn by building a culture for remote work.

# Meeting the team

I've been lucky to meet my team in Barcelona at two occasions: once in January for a company meeting, and once in March for a team building event at Port Aventura outside of Barcelona.

I realized I am no longer the young one in the team. In fact, I'm generally older than most at the company, which was a strange experience. There was at least one occurrence of where had to state "You go have fun on that ride, I will keep still at the ground" at Port Aventura.

# Technicalities

On the technical side, I've been engaged in developing a major feature and changes in the foundations for our products. 

One was the ability for a third party device (phone) to deliver configuration updates to an offline device, which required changes to both firmware, backend and the mobile app. Another was to train a neural net to estimate battery capacity.

For a good part of my time, I've been working on [trouble](https://github.com/embassy-rs/trouble), a Bluetooth Low Energy (BLE) host stack in Rust that runs on multiple controller implementations (though my primary focus is Nordic nRF). I've gotten positive feedback on that, and others in the community are showing interest and contributing fixes. I will write more about trouble in a future post.

# Summary

All in all it feels like I've learned a ton of new things since I joined Akiles. I think the coolest thing I've experienced here is the short path from implementation to production, for good and bad. It means I can very easily wreak havoc, but at the same time seeing firmware with code I wrote rolling out to all the devices gives a nice feeling. I'm looking forward to meet the team again, and the new things I'll be working on going forward, so stay tuned for more.

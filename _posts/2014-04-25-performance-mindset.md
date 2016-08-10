---
layout: post
title: "Performance mindset"
author: Ulf Lilleengen
categories: programming performance technical
---
You've all heard it, and you all know it: premature optimizations are the root of all evil. But what does that mean? When is an optimization premature? I've
come to think of this sort of 'dilemma' many times at work, where I see both my self and coworkers judging this by different standards.

Some programmers really don't care about performance at all. They write code that looks perfect, and fix performance issues as they come. Others think about performance all the time, where they will happily trade good software engineering principles for improved performance. But I think that thinking about performance in both of these ways are wrong. Performance is about simple and 'natural' design.

Let me give one recent example from work. A few months back, we rewrote parts of our software to get new features. Most importantly this involved a new protocol to be used between some client and server. After a while, users of our software were complaining about a server process consuming an unreasonable amount of memory. After profiling the server in question, we found that they were right: there was no need for that process to use that much memory. While profiling, we found a lot of things consuming an unecessary amount of memory, but the most important of them was that some large piece of data was copied many times before sending it through the cable.

Why did we write software like that? Well, while writing the new protocol we thought that it didn't matter. The data was typically not that big, and the server could handle the load fairly well according to benchmarks. Turns out that the user was serving much more data through that server than we anticipated. Luckily, increasing the JVM heap size made things go around, so it was no catastrophe.

We set off to fix it, and the amount of 'garbage' created was much less and memory usage improved significantly. But to get there, we had to introduce a new version of our protocol and refactor code paths to be able to pass data through the stack without copying. It was no more than 5 days of work, but those days could probably have been spent doing more useful things.

But surely, you'd expect the code to become horrible, full of dirty hacks and as unmaintainable as OpenSSL? Actually, it didn't. The refactorings and protocol changes did not worsen the state of our software. In fact, the design became better and more understandable, because we had to think about how to represent the data in a fashion that reduced the amount of copying done. This gave us an abstraction that was easier to work with.

Premature optimizations are really evil if they make your software harder to read and debug. Optimizations at the design level on the other hand, can save you a lot of trouble later on. I am under the impression that a lot of software is written without a 'performance mindset', and that a lot of manpower is wasted due to this. If we used 1 day to properly think through our protocol and software design in terms of data flow, we could spend 4 days at the beach drinking beer.

I like beer.

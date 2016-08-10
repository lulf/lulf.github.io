---
layout: post
title: "Exam reading"
author: Ulf Lilleengen
categories: personal school
---
Damn, reading for exams is really not my favorite thing. It's not that it's very
hard material, but the motivation is the problem. I always tend to get a bit
sloppy with classes where the only form of assessment is the exam, and if the
class is not very interesting either, it gets hard. However, these kind of
classes are typically very theoretical courses, and one way I cope with it is to
make them practical. For instance, in this course there are lot of distributed
algorithms that the student is expected to know. Some of them are almost several
pages long, and I'm really not the type for keeping all that in my head, and if
I did, it would only be because I memorized it. So instead, I tried to implement
the algorithm, as it helps with understanding because you can see
how it works in action! What I did in this case was to create a node
abstraction/class which I could re-use in several algorithms. The nodes
definition is something like this:

	void send(Message, nodeid);	// Send message to a single node
	void multicast(Message);	// Multicast message to all neighbouring nodes
	void deliver(Message);		// RMI method called by other nodes via their send method
	Message receive();		// Blocking receive method to fetch contents from buffer

The node creation itself adds necessary neighbours, and connections are
specified at startup time. The Message class contains most info necessary, but
is extended in some algorithms that need extra stuff. I implemented these
algorithms using the interface:

* Ricart-Agrawala's mutex algorithm
* Maekawas mutex algorithm
* Peterson election in unidirectional ring

Some algorithms are really tricky, and I end up spending more time wondering how
to implement it than actually doing it, so I guess this technique is not good
always :)

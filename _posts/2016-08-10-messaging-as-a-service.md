---
layout: post
title:  "Messaging as a Service"
author: Ulf Lilleengen
categories: technical enmasse
---
Inspired by a great [blog post](http://blog.effectivemessaging.com/2016/08/scalable-amqp-infrastructure-using.html) by Jakub Scholz on "Scalable AMQP infrastructure using Kubernetes and Apache Qpid", I wanted to write a post about the ongoing effort to build Messaging-as-a-Service at Red Hat. Messaging components such as the [Apache Qpid Dispatch Router](https://qpid.apache.org/components/dispatch-router/index.html), [ActiveMQ Artemis](https://activemq.apache.org/artemis/) and [Qpidd](https://qpid.apache.org/components/cpp-broker/index.html) scales well individually, but scaling a large deployment can become unwieldy. As Scholtz demonstrates, there are a lot of manual setup when creating such a cluster using kubernetes directly.

The [EnMasse](https://enmasseproject.github.io) project was created to provide the required tools and services for deploying and running a messaging service on [Openshift](https://www.openshift.com/). Running on Openshift means you can either run EnMasse on your own instance or in the cloud. You can also run EnMasse on [Openshift Origin](https://www.openshift.org/), which is the upstream community project. The long term goals with this project is to build a messaging service with the following properties:

   * Different communication patterns like request-response, pub-sub and events
   * Store-and-forward semantics
   * Support for a variety of different protocols like AMQP, MQTT, HTTP(1.1 & 2), CoAP and STOMP
   * Multi-tenancy
   * Scalability
   * Elasticity without disruption

The rest of this post will try to give an initial overview of EnMasse. EnMasse is still under
development, so a lot of the features mentioned may not be implemented or they are work in progress.

EnMasse can be configured with a list of addresses. Each address can have 4 different semantics in EnMasse:

   * Anycast: Messages go from a client, through the router network, to another client connected to the router network on the same address.
   * Broadcast: Messages go from a client, through  the router network to all clients connected to the router network on the same address.
   * Queue: Messages go from a client to a queue. Another client can read the message from the queue at a later point.
   * Topic: Aka. pub/sub. Messages go from a publisher client to multiple clients subscribed to the same address.

EnMasse is composed of the router network, the broker clusters, and the cluster admin components. The router network and the broker clusters handle messages, while the cluster admin components handles the router and broker configuration.

![EnMasse Overview]({{ site.url }}/images/enmasse_overview.png)

EnMasse contains two configuration files, address and flavor. The configuration files are stored as JSON within openshift as configmaps. The flavor configuration contains the supported variants of broker and router configurations. The address configuration contains the addresses the cluster should be able to handle and their desired semantics such as store-and-forward, multicast and the flavor type. The intention is that the cluster administrator is responsible for the available flavors, while the developer only has to care about which addresses to configure.

The main components of EnMasse are the router, broker and the cluster admininistration components.

### Router

EnMasse uses the Apache Qpid Dispatch Router to scale the service in terms of the number of connections it can handle as well as the throughput.  The router also hides the brokers from the client so that the brokers themselves may be scaled, moved, upgraded and changed without the client noticing.

### Broker

EnMasse creates broker clusters for queue and topic addresses. At present, EnMasse only supports ActiveMQ Artemis (or Red Hat JBoss A-MQ) as the message broker, though other brokers might be supported in the future. The brokers can also be scaled in the same way as routers, and EnMasse will ensure the cluster is configured correctly.

### Cluster administration

EnMasse contains several cluster administration components that manages the router and broker configuration:

   * The configuration service provides a way for the router agent and the subscription service to subscribe for a list of all addresses configured in EnMasse.
   * The router agent is responsible for configuring the network of routers based on the address configuration.
   * The subscription service is responsible for managing durable subscriptions so that a reconnecting client will transparently connect to the same broker.
   * The storage controller is responsible for creating, reconfiguring and deleting broker clusters based on two configuration files stored as openshift config maps. This component can be omitted, but will ease the maintenance when you want to configure multiple addresses.


I will try to write more articles about EnMasse as we are progressing with new features and improvements. For now, you can easily get started by following the [github example](https://github.com/EnMasseProject/openshift-configuration#setting-up-enmasse).  Do not hesitate to report bugs, and all contributions are always welcome.

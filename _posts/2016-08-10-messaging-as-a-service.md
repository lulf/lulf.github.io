---
layout: post
title:  "Messaging as a Service"
author: Ulf Lilleengen
categories: technical enmasse
---
Inspired by a great [blog post](http://blog.effectivemessaging.com/2016/08/scalable-amqp-infrastructure-using.html) by Jakub Scholz on "Scalable AMQP infrastructure using Kubernetes and Apache Qpid", I wanted to write a post about the ongoing effort to build Messaging-as-a-Service at Red Hat. Messaging components such as the [Apache Qpid Dispatch Router](https://qpid.apache.org/components/dispatch-router/index.html), [ActiveMQ Artemis](https://activemq.apache.org/artemis/) and [Qpidd](https://qpid.apache.org/components/cpp-broker/index.html) scales well individually, but scaling a large deployment can become unwieldy. As Scholtz demonstrates, there are a lot of manual setup when creating such a cluster using kubernetes directly.

The [EnMasse](https://enmasseproject.github.io) project was created to provide the required tools and services for deploying and running a messaging service on [Openshift](https://www.openshift.com/). Running on Openshift means you can either run EnMasse on your own instance or in the cloud. You can also run EnMasse on [Openshift Origin](https://www.openshift.org/), which is the upstream community project.

The rest of this post will try to give an initial overview of EnMasse.

![EnMasse Overview]({{ site.url }}/images/enmasse_overview.png)

EnMasse contains two configuration files, address and flavor. The configuration files are stored as JSON within openshift as configmaps. The flavor configuration contains the supported variants of broker and router configurations that. The address configuration contains the addresses the cluster should be able to handle and their desired semantics such as store-and-forward, multicast and the flavor type. The intention is that the cluster administrator is responsible for the available flavors, while the developer only has to care about which addresses to configure.

The main components of EnMasse are the router, broker and the cluster admininistration components.

### Router

The Apache Qpid Dispatch Router allows the service to scale by adding more of them using builtin kubernetes features. The router also hides the broker from the client so that the brokers can be scaled, moved, upgraded and changed without the client seeing.

### Broker

To support store-and-forward semantics, brokers such as ActiveMQ Artemis is used. The brokers can also be scaled in the same way as routers, and EnMasse will ensure the cluster is configured correctly.

### Cluster administration

In addition to these well known components, EnMasse contains several cluster administration services that manages the router and broker configuration:

   * Configmap-bridge provides a way to subscribe for configmap updates through AMQP.
   * Router agent is responsible for configuring the network of routers based on the address configuration.
   * Storage controller is responsible for creating, reconfiguring and deleting broker clusters based on the address and flavor configuration.


I will try to write more articles about EnMasse as we are progressing with new features and improvements. For now, you can easily get started by following the [github example](https://github.com/EnMasseProject/openshift-configuration#setting-up-enmasse).  Do not hesitate to report bugs, and all contributions are always welcome.

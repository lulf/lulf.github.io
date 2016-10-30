---
layout: post
title: Cluster management using Kubernetes and OpenShift
author: Ulf Lilleengen
categories: technical openshift docker kubernetes cloud
---

Docker containers have become popular means of packaging and running software. In a cloud
environment, containers makes deploying software easy both for developers and system adminstrators.
In fact, is has become so easy that the roles have started to merged, creating devops.

Large technology companies like Google, Facebook, Yahoo and others have made a significant
investement in automating their process. At their scale, this is absolutely necessary in order to
deliver new features. For smaller companies, the gains have not always been worth the effort.

Luckily, open source software is now running the world. We have seen software for managing large
scale distributed systems created and open sourced. Kubernetes, OpenShift, Apache Mesos and Docker
Swarm are all examples of this, and this article will focus on Kubernetes and OpenShift. But first, a short look at the evolution of software packaging.

In the beginning was the tarball. It contained the software to distribute, along with a few scripts
to build and help with installing. It could also contain the software in binary forms built for a
specific platform. This way of packaging is still used today, and has found a use as a mechanism to
distribute artifact bundles in CI and CD systems.

The biggest downside of tarballs is that there is no metadata and dependency management. The
dependencies are assumed installed on the target system. The package managers and package format
solves this problem by combining the software with metadata describing its assumptions about the
target system. 

Packages are typically built and distributed as part of a Linux distribution release. In the recent
years, developers require newer versions of software than what is part of a distribution. Although
distributions have shortened their release cycles, the respective support provided for updates and
security fixes is also shortened.

With docker containers, we are kind of getting a mix of tarballs and packages. It allows sharing
dependencies through unionfs and provides metadata, but choose your own way of building your
software.

Imagine for a second that all software you installed on your computer was in the form of docker
containers. Then imagine upgrading your system and keeping images up to date. 

Now how does this relate to cluster management? Well, with continous deployment, the distribution of
software and run time management of the service is coupled. 

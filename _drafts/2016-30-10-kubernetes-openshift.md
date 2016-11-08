---
layout: post
title: Cluster management using Kubernetes and OpenShift
author: Ulf Lilleengen
categories: technical openshift docker kubernetes cloud
---

Docker containers have become popular means of packaging and running software. In a cloud
environment, containers makes deploying software easy both for developers and system adminstrators.
In fact, is has become so easy that the roles have started to merged, creating devops.

# Cluster management - nothing new

Cluster management is usually the task of managing and configuring services and processes in a
distributed system. Ever since the cloud made its entry, cluster management become increasingly
important. Nodes, an abstract term for a machine, can come and go frequently, and virtualization has
made managing nodes and applications easier.

Anyone developing or maintaining a service constiting of different standalone components will at
some time develop some form of cluster management. It starts out as a set of shell scripts at first, then evolving into
some additional components in the system that will keep track of the topology and state.

Large technology companies like Google, Facebook, Yahoo and others have made a significant
investement in automating their deployment process and cluster management facilities. At their scale, this is absolutely necessary in order to deliver new features. For smaller companies, the gains have not always been worth the effort.

Luckily, open source software is now running the world. We have seen software for managing large
scale distributed systems created and open sourced. Kubernetes, OpenShift, Apache Mesos and Docker
Swarm are all examples of this, and this article will focus on Kubernetes and OpenShift. But first,
a short look at the evolution of software packaging.


# Software packaging

The tarball is one of the most popular way to package software. It typically contains the software
itself, along with a few scripts to build the source code and/or help with installing the software. 

The biggest downside of tarballs is that there is no metadata and dependency management. The
dependencies are assumed to be installed on the target system. The package managers of various Linux distributions and their package formats solve this problem by combining the software with metadata describing its dependencies and assumptions about the target system.

Packages are typically built and distributed as part of a Linux distribution release. In the recent
years, there has been a trend for doing rapid releases, and the packages in a distribution quickly
become outdated.  Although distributions have shortened their release cycles and some packages are available through third party repositories, programming languages themselves now contain their own package managers that developers use. So, on one hand you have the system administrator who wants to be in control of whatever packages are installed on the host. On the other hand, you have the developer A who needs one set of dependencies, and developer B who needs a different set of dependencies.

Docker containers allows developers to choose how their application dependencies are installed
independent of the host system. Moreover, it provides the appropriate abstractions satisfying
requirements both from developers and system operators. Therefore, container images can be seen as a
'unit of distribution', and this is indeed the assumption in Kubernetes and OpenShift.

# Kubernetes

Now that container images have become the unit of distribution, it allows for creating generic
cluster management functionality.

Kubernetes is the third generation of cluster management made at Google, and this time they made it
generic without bindings to Googles own infrastructure. Kubernetes is the second most active github
project, with Red Hat as the second largest contributor. The reason for that is OpenShift, which
I will come back to later in this post.

The kubernetes overall architecture is shown below.

INSERT IMAGE HERE



# Openshift

# Example

# References

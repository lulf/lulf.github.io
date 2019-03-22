---
layout: post
title:  "Gitops and EnMasse"
author: Ulf Lilleengen
categories: technical kubernetes enmasse gitops openshift
---

With the EnMasse 0.27.0 release, there is a cool feature that enables a nice gitops integration for
messaging applications. 

Gitops is a way to do Continuous Delivery where all configuration and state for an application is stored in git, and changes to an application is done as pull/change requests to that git repo. Once the PR has been tested and reviewed, it can be merged. When merged, a CD job is triggered that will apply the current state of the git repository to the system.

The gitops model fits well with how Kubernetes works, because you can store your applications
Kubernetes configuration in git, and trigger some process to apply the configuration to a
Kubernetes cluster. If you store your application code together with the Kubernetes configuration,
you enable development teams to be in full control of their application deployment to any cluster environment.

Traditionally, Kubernetes has mainly been used for stateless services. Stateful services has normally been run outside the Kubernetes cluster.
If a development team wants to use a stateful service on the cluster, the team normally have to install and manage the service themselves.

However, with EnMasse, the responsibility of operating the messaging service is separate from the tenants consuming it, and you can separate the concern of operating the messaging service from consuming it. This makes it easy for development teams to use the gitops model with on-cluster services managed by someone else.

Lets assume that you have a team in your organization managing the messaging infrastructure using EnMasse on Kubernetes or OpenShift.

From the development team point of view, the following manifests must be declared:

* The messaging configuration
* The application configuration

The messaging configuration consists of the AddressSpace, Address and MessagingUser resources.

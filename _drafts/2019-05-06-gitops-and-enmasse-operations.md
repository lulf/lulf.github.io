---
layout: post
title:  "Gitops and EnMasse - Operations deep dive"
author: Ulf Lilleengen
categories: technical kubernetes enmasse gitops openshift monitoring
---

With the EnMasse 0.28.0 release, using a [Gitops](https://www.weave.works/technologies/gitops/) workflow to manage your messaging application is even easier than before. This article is a followup on [Gitops and EnMasse](/technical/kubernetes/enmasse/gitops/openshift/2019/04/08/gitops-and-enmasse.html) with focus on the operations side of things. I recommend that you read that article first to get an overview of gitops and EnMasse in general.

So as in the previous article, lets assume that you have a team in your organization managing the messaging infrastructure using EnMasse on Kubernetes or OpenShift, and that you have 2 independent developer teams that both want to use messaging in their applications. The following diagram describes the flow: 

![Gitops]({{ site.url }}/images/enmasse_gitops_operations.png)

The operations team will deploy the messaging infrastructure (EnMasse), and commit the desired configuration templates that they want to support to git. A CI process then applies the EnMasse configuration to the cluster.

In this article, we will start with an EnMasse release, remove the bits we don't need, and apply configuration specific to the service we are going to offer. We want to provide the following:

* The ability to provision brokers on-demand for multiple development teams
* The ability for development teams to manage authentication and authorization policies for their messaging applications
* Restricting the v


# Installation

Managing an EnMasse deployment in git can be as simple as unpacking the [release](https://github.com/EnMasseProject/enmasse/releases) bundle and committing the parts that is used for a particular installation. 

First, unpack the latest release (0.28.0 at the time of writing):

```
mkdir myservice && cd myservice
wget https://github.com/EnMasseProject/enmasse/releases/download/0.28.0/enmasse-0.28.0.tgz -o enmasse.tgz
tar xvf enmasse.tgz
```

EnMasse can be installed either YAML files or Ansible. In this guide we will use the YAML, and
remove the bits that we don't need and commit

```
git init
rm -rf enmasse/ansible
rm -rf enmasse/docs
git add enmasse
git commit -a -m 'Initial import'
```



# Configuration

# Monitoring


In addition, the operations team must apply some service configuration 

## Summary

We have seen how an operations team and a set of development teams can manage messaging as Kubernetes manifests. This allows your whole organisation to follow the gitops model when deploying your applications using messaging on Kubernetes and OpenShift. 

Star the project on [github](https://github.com/EnMasseProject/enmasse/) and follow on [twitter](https://twitter.com/enmasseio)!

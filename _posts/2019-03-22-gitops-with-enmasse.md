---
layout: post
title:  "Gitops and EnMasse"
author: Ulf Lilleengen
categories: technical kubernetes enmasse gitops openshift
---

With the EnMasse 0.27.0 release, using a [Gitops](https://www.weave.works/technologies/gitops/)
to manage your messaging application is even easier than before.

Gitops is a way to do Continuous Delivery where all configuration and state for an application is stored in git, and changes to an application is done as pull/change requests to that git repo. Once the PR has been tested and reviewed, it can be merged. When merged, a CD job is triggered that will apply the current state of the git repository to the system.

The gitops model fits well with how Kubernetes works, because you can store your Kubernetes configuration in git, and trigger some process to apply the configuration to a Kubernetes cluster. If you store your application code together with the Kubernetes configuration, you enable development teams to be in full control of their application deployment to any cluster environment.

Traditionally, Kubernetes has mainly been used for stateless services. Stateful services has normally been run outside the Kubernetes cluster.
If a development team wants to use a stateful service, the team normally have to install and manage the service themselves or use a cloud provider service.

However, with EnMasse, the responsibility of operating the messaging service is separate from the tenants consuming it, and you can separate the concern of operating the messaging service from consuming it. This makes it easy for development teams to use the gitops model with on-cluster services managed by someone else.

Lets assume that you have a team in your organization managing the messaging infrastructure using EnMasse on Kubernetes or OpenShift, and that you have 2 independent developer teams that both want to use messaging in their applications.

The operations team will deploy the messaging infrastructure, and apply the desired configuration templates that they want to support. 

The development teams will deploy their application resources along with the messaging resources needed by the application.

The messaging resources consists of the AddressSpace, Address and MessagingUser.

An AddressSpace is a group of addresses that share connection endpoints as well as authentication and
authorization policies, and is defined as this:


```
apiVersion: enmasse.io/v1beta1
kind: AddressSpace
metadata:
  name: myspace
  namespace: team1
spec:
  type: standard
  plan: standard-small
  endpoints:
  - name: messaging
    service: messaging
    cert:
      provider: openshift
    exports:
    - name: messaging-config
      kind: ConfigMap
```

Messages are sent and received from an address. An address has a type that determines its semantics, and a plan that determines how much resources
is reserved for this address. An address can be defined like this:

```
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: myspace.queue1
  namespace: team1
spec:
  address: queue1
  type: queue
  plan: standard-small-queue
```

To ensure that only trusted applications are able to send and receive messages to your addresses, a
messaging user must be created. For applications running on-cluster, you can authenticate clients
using a Kubernetes service account. A serviceaccount-authenticated user can be defined like this:

```
apiVersion: user.enmasse.io/v1beta1
kind: MessagingUser
metadata:
  name: myspace.app
  namespace: team1
spec:
  username: system:serviceaccount:team1:default
  authentication:
    type: serviceaccount
  authorization:
  - operations: ["send", "recv"]
    addresses: ["queue1"]
```

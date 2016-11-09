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

Now that container images have become the unit of distribution, it simplifies the work of creating a generic
cluster management functionality, which exactly what Google has done.

Kubernetes is the third generation of cluster management made at Google, and this time they made it
generic without bindings to Googles own infrastructure. Kubernetes is the second most active github
project, with Red Hat as the second largest contributor. The reason for that is OpenShift, which
I will come back to later in this post.

The kubernetes overall architecture is shown below.

![Kubernetes overview]({{ site.url }}/images/kubernetes_overview.png)

A host in k8s can have 2 roles: master and node. Masters typically (but don't necessarily have to)
run the central database with the state of all resources in the kubernetes instance. Before diving
into what the master does, lets take a look at the nodes first.

A node contains an agent process that communicates with the master. The agent process is responsible
for starting, stopping and configuring pods. A pod is a basic schedulable entity in k8s that may
have 1 or more containers. This means that a pod in principle can be scheduled to run on any node in the cluster, but this can be restricted through the use of labels. All resources (nodes, pods, etc.) can be labeled, which allows all resources to be looked up using a label selector. 

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: container1
      image: mydocker/image:latest
      ports:
        - name: main
          containerPort: 8080
      volumeMounts:
        - name: data
          mountPath: /var/db/app_volume      
  volumes:
    - name: data
      emptyDir:
```

As you can see in the pod specification, it specifies a container and a set of ports. In addition, the pod specifies a volume that it mounts inside the container. A volume can be backed in many ways:

   * emptyDir
   * hostPath
   * gcePersistentDisk
   * awsElasticBlockStore
   * nfs
   * iscsi
   * flocker
   * glusterfs
   * rbd
   * cephfs
   * gitRepo
   * secret
   * persistentVolumeClaim
   * downwardAPI
   * azureFileVolume
   * azureDisk
   * vsphereVolume
   * Quobyte
    
I won't go into the different types, but one interesting one is the persistentVolumeClaim type. This mechanism decouples the underlying type of volumes from the claim. This means that volume claims can be created as a resource by an administrator, and a pods may refer to this claim when specifying its volumes. The volume may also refer to a disk exposed by the host system, allowing you to run applications that require high performance local storage.

Lets have another look at the master. The above specification is sent to the master using the kubectl command line tool or the k8s REST API. The master creates the resource in its database, and the scheduler will locate the appropriate node for running the node. The picture below shows a more detailed view of the k8s architecture.

![Kubernetes details]({{ site.url }}/images/kubernetes_overview_detail.png)

Here you can see that the master contains some basic authentication and authorization for external REST clients and a scheduler as mentioned above. The master also runs different controllers, like a replication controller. 

The replication controller manages replicas of a pod. It is specified as a resource in a simpliar fashion as a pod:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-controller
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: container1
          image: mydocker/image:latest
          ports:
            - name: health
              containerPort: 8080
          volumeMounts:
            - name: data
              mountPath: /var/db/app_volume      
      volumes:
        - name: data
          emptyDir:
```  

This specification will ensure that there exists 3 pods with the given specification. Moreover, if one of the pods crashes, it will spin up a new instance of that pod. Replication controllers and pods are only two of many types of resources in a k8s cluster, and there are other types as well that can meet your requirements:

* Jobs
* Deployments
* PersistentVolume
* Service
* Secret
* ConfigMap

# Openshift

Open

# Example

# References

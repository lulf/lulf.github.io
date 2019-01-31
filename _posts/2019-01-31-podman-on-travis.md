---
layout: post
title:  "Building containers on Travis CI using Podman"
author: Ulf Lilleengen
categories: technical kubernetes podman travis ci
---

Since [Podman](https://podman.io) and [Buildah](https://github.com/containers/buildah) appeared on my radar, I've been wanting to try replacing docker. Podman is a replacement for docker, whereas buildah is a replacement for docker build. Although docker works OK, I've seen various issues with different versions of docker not working with Kubernetes and OpenShift, and that the local docker daemon sometimes becomes unresponsive and causes build failure in the [EnMasse](http://enmasse.io) CI. Since podman and buildah does not use a local daemon for building images, they will work without root privileges. 

The main difference between podman and buildah from a user perspective is that the podman has a wider feature set than buildah, and the podman cli is almost 1:1 with docker. Podman also has the ability to run containers and generate Kubernetes manifests, whereas buildah is focused only on building container images.

Using podman on [Travis CI](https://travis-ci.org) is somewhat a challenge, for a few reasons:

* Podman is not available in the default Ubuntu repositories and a newer version of Ubuntu than the
  default travis one is needed
* Podman assumes a Fedora/CentOS/RHEL container configuration (/etc/containers/registries.conf)

So to replace docker with podman, ensure you have the following set in your `.travis.yml`:

```
dist: xenial
before_install:
- sudo add-apt-repository -y ppa:projectatomic/ppa
- sudo apt-get update -qq
- sudo apt-get install -qq -y software-properties-common podman
- sudo mkdir -p /etc/containers
- sudo sh -c 'echo -e "[registries.search]\nregistries = [\"docker.io\"]" > /etc/containers/registries.conf'
```

The last 2 lines are necessary for podman to be able to fetch images from Docker Hub.

In your build scripts, you can replace `docker` with `podman`, and thats it!

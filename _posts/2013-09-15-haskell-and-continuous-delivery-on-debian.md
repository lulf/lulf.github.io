---
layout: post
title: "Haskell and continuous delivery on debian"
author: Ulf Lilleengen
categories: haskell programming web yesod continuous delivery deployment debian
---

Having run my toy web service [wishsys](http://wishsys.dimling.net) for a few months, I thought it would be nice to setup a [continuous](http://en.wikipedia.org/wiki/Continuous_delivery) delivery/deployment pipeline for it, so that I could develop a new feature and push it into production as quickly as possible (if it passes all tests). I thought writing a small article about this would be nice as well, as I found few resources for doing CD with haskell.

The wishsys code is written in [haskell](http://www.haskell.org/haskellwiki/Haskell) and uses the [yesod](http://www.yesodweb.com/) web framework. Update: I noticed a comment on reddit mentioning [keter](https://github.com/snoyberg/keter), which is essentially what I should have used instead of debian packages since I'm using yesod. This guide should still be relevant regarding CI though.

## Choosing CI system

After doing a little research, I ended up with two alternatives to use for CI.

* [Jenkins](http://jenkins-ci.org/)
* [Travis](https://travis-ci.org/)

I have some experience with Jenkins at work, and since its very generic, I thought it would be easy to build a haskell project with it. Since the wishsys code is hosted at [github](https://github.com/lulf/wishsys), I was tempted to try travis, since it integrates well with github. I decided to go with Jenkins instead, mainly because I had a VPS to run it, and I didn't need the great Heroku support, as I would be deploying it to the same VPS. There is a good introduction of CI for Haskell and Yesod at [yesodweb](http://www.yesodweb.com/blog/2012/10/haskell-and-ci).

## Git and branches

Having chosen which CI-system to use, I needed to structure the git repository in a way that fits my workflow.
Since I wanted to have some control over what changes that are pushed out to production, I created a stable branch in the repository, which is the branch from which the production binaries are compiled. All development goes into other branches. For CI to have any meaning, all of these branches should be built all the time so I can be sure that my tests are stable and that they pass. The plan was to create a build job for every branch that builds and tests that particular branch.


## Setting up Jenkins
Setting up Jenkins is very easy on Debian, as you just need to add the [jenkins debian repo](http://pkg.jenkins-ci.org/debian/) and install packages. Having done that before, I just started on creating a new project for wishsys. Jenkins has a lot of plugins, and since this is a github project, I installed the github plugin for jenkins as well.

Having only the stable branch, I created one job called wishsys-build-stable. To build haskell, I added an entry under "Execute shell", which runs the following commands when building:

    $CABAL sandbox init
    $CABAL --enable-tests install

$CABAL is parameterized to /var/lib/jenkins/.cabal/bin/cabal since I needed a newer cabal version to get the sandbox functionality. I also configured the job to trigger for every commit, though I will probably change it to build as often as possible to ensure there are no unstable tests.


## Creating a debian package

The next step was to create debian packages of the software, so I could easily install it in my VPS (manually or automatically). Since this was unfamiliar territory, it took some time and reading to grasp the package layout, but I found an [intro guide](https://wiki.debian.org/IntroDebianPackaging) for creating debian packages that helped me. In addition, I added a [makefile](https://github.com/lulf/wishsys/blob/stable/Makefile) that invokes all of the cabal commands to build my project and to do what the debian package tools expect.

## Creating debian packages in jenkins
I then started looking for a jenkins plugin to help me build the debian package, and found the following plugins:

* [debian-package-builder](https://wiki.jenkins-ci.org/display/JENKINS/Debian+Package+Builder+Plugin)
* [jenkins-debian-glue](http://jenkins-debian-glue.org/)

debian-package-builder seemed to only support automatic version manipulation for subversion repositories. Since wishsys uses git, I went for jenkins-debian-glue. Creating a debian package is complicated in itself, and I initially spent some time doing a lot of what jenkins-debian-glue tries to do automatically (automatically creating a tarball of the repo and running git-import-orig and git-buildpackage).

I used [this](http://jenkins-debian-glue.org/getting_started/manual/) guide to setup the jenkins jobs. I ended up with a wishsys-debian-source job for building the source package, and a wishsys-debian-binaries job for creating binary packages. 

The jobs are run in the following order: wishsys-build-stable -> wishsys-debian-source -> wishsys-debian-binaries

The wishsys-build-stable job is run for each commit, and reuses the git checkout between builds to reduce build times. The wishsys-debian-source job simply creates a tarball of the git repo, and forwards the resulting tarball to the wishsys-debian-binaries job, which does a full clean build of the software before creating the binary package itself.

## Summary

Setting up a CD pipeline is a great way to get your features tested and into production in minimal time. Though this setup is somewhat debian specific, the generic pattern should be reusable. In the future, I would like to avoid building the project twice (once in wishsys-build-stable, and once in wishsys-debian-binaries), but the build times are currently not an issue. Another improvement would be to get hunit and quickcheck test reports displayed in Jenkins.

The set of files necessary for debian build are available at [github](https://github.com/lulf/wishsys/tree/stable/debian).



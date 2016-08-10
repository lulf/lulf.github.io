---
layout: post
title: "Wishing with yesod"
author: Ulf Lilleengen
categories: haskell programming web yesod
---
Today, I launched my [wishsys](http://wishsys.dimling.net) service, which is just a simple service for creating wish lists with separate access for owners and guests. The original use case was my own wedding, so I created an even simpler version for that using [snap](http://snapframework.com/). Snap worked great, but I had some hassle building the authentication mechanisms properly. 

After the wedding, one of the guests wanted to use the same system for their wedding, so I thought I might as well create a more generic wish list service. This time, I went with [yesod](http://www.yesodweb.com/), as it seemed to provide more of a platform than a framework.

At Yahoo!, I work on a search platform, and there are a few things I expect from a platform. It should provide

* Storage
* Access control
* Higher level APIs for request handling
* Internalization
* Good APIs
* Good documentation
* Ease of deployment
* Test framework

Yesod did not let me down. It provides a real [book](http://www.yesodweb.com/book), and not just a bunch of outdated wiki pages. Its solution for storage is excellent. [Persistent](http://www.yesodweb.com/book/persistent) allows me to write the definition of my data structures in a single place, and automatically generate a database schema and haskell types. I chose to use postgresql as my persistent backend, and by using the scaffold code, getting it working was trivial. Creating request handlers was so easy, I won't even tell you how I did it.

My biggest yesod issue was [authentication](http://www.yesodweb.com/book/authentication-and-authorization), since I had somewhat special requirements where I wanted to have two users with different access levels (admin and guest). I also missed a method in the authentication system to request a user to be logged in, regardless of what authentication backend used. I ended up looking at what HashDB did internally, and just copy that (If there is a better way, please let me know).

I used the [hamlet template system](http://www.yesodweb.com/book/shakespearean-templates) to write HTML with minimal haskell clutter. [Forms](http://www.yesodweb.com/book/forms) are a pleasure to work with, because I don't have to repeat myself. I just had to create the form in one place, and I could then use it both for generating correct HTML and easily parse the POST request.

I just followed the [deployment](http://www.yesodweb.com/book/deploying-your-webapp) chapter when deploying, and then the service was suddenly live. Even more important to note is the development server, which automatically compiles the app if something changes. Great for local testing!

My biggest issue with yesod was understanding compilation errors messages. But, when I got things working, yesod was a great experience. It is one of the few open source projects I've seen that understands what it means to be a platform, and it thinks of your needs before you realize them. Kudos!

Btw, the wishsys source code can be found on [github](https://github.com/lulf/wishsys)

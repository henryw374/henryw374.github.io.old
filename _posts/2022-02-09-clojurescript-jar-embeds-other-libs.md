---
layout: post
title: Clojurescript jar embeds some core libraries
description: Why having clojurescript in the classpath may lead to unexpected behaviour
category: clojure
---

The [clojurescript maven artifact](https://mvnrepository.com/artifact/org.clojure/clojurescript/1.11.4) 
lists compile dependencies which include: data.json, tools.reader and transit-clj and transit-java. 

However the clojurescript jar itself is something like an uberjar: It includes compiled data.json, tools.reader and transit-clj and transit-java
namespaces inside itself. 

The effect is that if you want to use a different version of one of those libraries 
compared to the one Clojurescript was compiled with, you can't. This was not an issue
for a long time because those libraries didn't change. Now e.g. clojure.data.json has
changed, hence why I hit the problem. 

One might ask why I would be using Clojurescript and clojure.data.json together in the same jvm. 
Well, in my case, in development I tend to have my server and client dependencies combined, so 
I run cljs compile and server side stuff in one vm. When deploying, testing I separate them 
(Clojurescript is not in the classpath). It is possible to run separate server and cljs jvm's 
locally of course, but that then means I can't have a single .nrepl.edn file for example. There could be 
other reasons for using these 2 together though, writing data-reader functions that use json possibly.  

I raised this on clojure slack and now Clojurescript's maintainer's are aware, so hopefully
this gets fixed. 

The fix is likely to involve `shading`. This is where a library wants to use a fixed version 
of another library, so it copies the sources of that library into itself, but changes 
the namespaces/packages of the source library to be something different, and specific to itself.

My thanks go to Alex Miller for explaining this.  
 
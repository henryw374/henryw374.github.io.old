---
layout: post
title: People are worried about Clojurescript build size?
description: A look at the trade-offs to be made when developing for the web, especially wrt date/time
category: clojure
---

I am going to compare the performance of two Clojurescript date-time libraries, in the context of 
a typical web application (SPA).

# The Libraries

## Cljc.java-time

[cljc.java-time](https://github.com/henryw374/cljc.java-time) has the same API as java.time, 
but targets both Clojure and Clojurescript. 
It is implemented on top of a pure Javascript implementation of java.time
                                            called JS-Joda.

It is also the underlying library for Juxt's [Tick](https://github.com/juxt/tick) date-time library
 which offers a powerful API beyond what java.time offers. In this blog post I am considering cljc.java-time
 instead of Tick, because I expect a larger proportion of readers will already have some familiarity 
 with cljc.java-time's API (which is the same as java.time). 

## Deja-fu

[Deja-fu](https://github.com/lambdaisland/deja-fu) is a new Clojurescript date/time library 
positioning itself as being for
"applications where dealing with time is not enough of their core business to justify these large dependencies".
The large dependendencies being referred to there are those required by cljc.java-time.

Deja-Fu's API offers a pure-cljs `Time` entity and otherwise wraps the platform js/Date objects, 
via the goog.date API.

[Cljs-time](https://github.com/andrewmcveigh/cljs-time) is similar to Deja-fu in that it wraps that same goog.date API.

# Motivation

In my own experience of developing Clojurescript web 
applications with cljc.java-time, I haven't see any performance problems - I'm already using Clojurescript and React so there's
already a significant build size, but given my users' devices and 
network connections (reasonably up to date, but nothing special) the applications seem to perform
very well.

I once did [a talk](https://www.youtube.com/watch?v=UFuL-ZDoB2U)
 introducing cljc.java-time and related libraries, but only briefly talked about build size - should I have said it 
 was only suitable
if date/time was so core to the app that "large dependencies" could be justified ? Build size [is already discussed](https://github.com/juxt/tick/blob/master/docs/cljs.adoc)
in the documentation to help users understand any likely impact. As developers we don't always have time to investigate all 
aspects of every candidate library we might use
and maybe someone would see the Deja-fu readme and choose that over cljc.java-time just because they feel 'time is not enough 
of their core business'. I feel these claims need to be looked at in more detail and quantified, 
so I decided
to do some experiments to try to provide a bit more colour around the relative costs of using these two libraries 
in the browser.
 
# The Experiment

For my experiment I have written two versions of a basic Clojurescript web application. 

People are using Clojurescript
in various places, including highly constrained environments like microcontrollers, but based on what 
I see the React webapp is what the majority are targeting and the use-case for which I would like people to have
some more help when choosing a date/time API. 

These apps have been deployed on the web so that tools such as [PageSpeed](https://developers.google.com/speed/pagespeed/insights/)
can be used to test them. They are hosted on Firebase, but just because I already had a dummy project set up there. They don't
use any Firebase APIs.

 Version             |  TTI  (mobile)        | TTI (desktop) | 
|---------------------|---------------|---------------|
| [Deja-fu version](https://friendly-eats-demo-e71b7.web.app/js-date.html) | 2.1s | 0.6s |
| [cljc.java-time version](https://friendly-eats-demo-e71b7.web.app/java-time.html) | 2.2s | 0.7s |

 TTI (time-to-interactive) shown in the table above was taken from PageSpeed analysis. The cljc.java-time version is slower
  according to that analysis, but not in any meaningful way.
  
Consider that (the Javascript behind) cljc.java-time and React are fixed-size costs though. Being a small demo app, they 
 are disproportionately big. If application code grows over time with features their 
relative size will reduce ofc.

The memory usage for both apps was roughly the same, as observed in a recent version of Chrome.

What about download size? Well, let's imagine that every time a user visits these apps, the Clojurescript code has 
been changed and released, so cannot be retrieved from cache and must be re-downloaded.
It will only take 2-3 visits before the total amount of data downloaded across those visits is greater 
in the Deja-fu version. This is because in the cljc.java-time version, the underlying JS dependencies are downloaded separately, 
and so are cacheable. This is very simple to set up and I would put it in the 'no brainer' category
if data allowance is a significant issue for an app. 

Maybe we could modularize the Deja-fu version so the library code can be cached over visits, 
but my main point here is YMMV.

Are there more metrics that we should look at here? Please suggest anything you think is significant.

# The code 

The [source code for these can be found here](https://github.com/henryw374/cljs-date-lib-comparison).

Shown below is the code that is different between the two versions.

## Deja-fu 

```clojure 

(ns time-lib-comparison.js-date
  (:require [lambdaisland.deja-fu :as deja-fu]
            [time-lib-comparison.app-main :as app]))

(def millis-per-day (* 1000 60 60 24))

(defn interval-calc [event-date]
  (let [now (deja-fu/local-date)
        event-date (deja-fu/parse-local-date event-date)
        interval-millis (- (deja-fu/epoch-ms event-date)
                          (deja-fu/epoch-ms now))]
    (/ interval-millis millis-per-day)))

(defn tomorrow []
  (-> (deja-fu/local-date)
      (update :days inc)))

```

### cljc.java-time 

```clojure 

(ns time-lib-comparison.java-time
  (:require [cljc.java-time.local-date :as date]
            [cljc.java-time.temporal.chrono-unit :as cu]
            [time-lib-comparison.app-main :as app]))

(defn interval-calc [event-date]
  (cu/between cu/days (date/parse event-date) (date/now)))

(defn tomorrow []
  (-> (date/now)
      (date/plus-days 2)))

```

# And now, a twist

Did you notice any bugs?

The libraries have been weighed up against each other performance-wise, but
of course that is only one part of the story. 

Based on my impressions of the Clojure community, I think that the majority of Clojurescript developers
 also develop in Clojure, on the jvm. That means that sooner or later they will need to use java.time as that is the 
 platform API. 
 Dates and times are complicated. What makes things needlessly more
difficult is having to use unfamiliar APIs. It's not just different method
signatures, but actual semantics. For example, if you 'add' a month to the 31st January, what happens?
A decision had to be made by the API authors, and that decision was made differently in java.time vs js/Date.

So I'll leave it to the reader to spot the bugs (I have deliberately put 2 in each version), but 
just offer a general word of warning that if you have
measured and found a worthwhile performance gain with some software you manage, make sure you consider 
the cost of that. Cost which 
might be measured in developer time and hard-to-spot bugs.

## Is this a fair test?

The main take-away I would hope readers
get from this is that that cljc.java-time should not be dismissed out of hand for some common use cases, just 
as we don't generally dismiss Clojure because it might be 'slower' or more 'memory hungry' compared to C. 

All that being said, I have chosen some requirements for the app where in the Deja-fu version I need to use to the much 
maligned js/Date API. 

Also, if the app needed to do custom parsing and formatting, for the cljc.java-time version I'd need to bring in a JSJoda addon, 
which takes TTI up to 2.5 seconds on mobile, whereas with the Deja-fu version TTI would be the same.

# Looking to the future

[Tempo](https://github.com/henryw374/tempo) is my work-in-progress attempt to make a date-time API with the common parts 
of java.time
and the new Temporal platform API for Javascript (to be available in browsers in sometime soon, possibly this year).

The fact that Temporal is a platform API is the big reason of course.

My feeling is there is sufficiently large overlap between Java and Javascript's platform date-time APIs to make a useful 
library that will suit cross-platform 
[library authors needing some basic date/time functionality such as Malli](https://github.com/metosin/malli/issues/49) and 
perhaps also as a basis for a 'lite' version of the Tick library. 

# Conclusion

There may well be Clojurescript applications where cljc.java-time would be inappropriate, but my feeling is 
for typical Clojurescript web applications it's not an issue.

I'd definitely be interested to hear about your opinions on this
and Clojurescript build sizes in general. What are you shipping? How did you make decisions 
about what build size was acceptable (including
the use of Clojurescript itself)?

Feel free to use [this thread on Clojureverse]() to comment. 
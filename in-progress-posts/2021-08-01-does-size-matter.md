---
layout: post
title: People are worried about Clojurescript build size?
description: A look at the trade-offs to be made when developing for the web, especially wrt date/time
category: clojure
---

# Intro 

It's been a few years since I started using JS-Joda (as Javascript implementation of java.time)
as the basis of some cross platform Clojure libraries and based on feedback, stars and downloads
there are plenty of people happily using these libraries. However, I have heard JS-Joda described a few times
as 'bloated' or 'just too big'. In my own experience of developing Clojurescript web 
applications I didn't see any problem - I'm already using Clojurescript and React so there's
already a significant build size, but given my users' devices and 
network connections (reasonably up to date, but nothing special) the applications seem to perform
very well. 

Recently a new Clojurescript date/time library [Deja-fu](https://github.com/lambdaisland/deja-fu) (based on Javascript's platform Date API) has come out, 
positioning itself as being for 
"applications where dealing with time is not enough of their core business to justify these large dependencies".
Large dependendencies here being JS date libraries. I did [a talk]()
introducing these libraries and only briefly talked about build size - should I have said they were only suitable
if date/time was core to the app? FYI Build size [is already discussed](https://github.com/juxt/tick/blob/master/docs/cljs.adoc)
in the documentation of Tick (which uses JSJoda). As developers we don't always have time to investigate all 
aspects of every candidate library we might use
and maybe someone would see the Deja-vu readme and choose that over e.g. [cljc.java-time](https://github.com/henryw374/cljc.java-time)
(which has the same API as java.time, but targets Clojure and Clojurescript) just because they know 'time is not enough of their core business'.
I feel like that would be a shame, so I decided
to do some experiments to try to provide a bit more colour on the cost of using JS-Joda.

# The Experiment

For my experiment I have written two versions of a basic Clojurescript web application, using React. People are using Clojurescript
in many various places, including highly constrained environments like microcontrollers, but based on what 
I see the React webapp is what the majority are targeting and the use-case for which I would like people to have
some help when choosing a date/time API. The [source code for these can be found here](https://github.com/henryw374/cljs-date-lib-comparison).

These apps have been deployed on the web so that tools such as [PageSpeed](https://developers.google.com/speed/pagespeed/insights/)
can be used to test them. They are hosted on Firebase, but just because I already had a dummy project set up there. They don't
use any Firebase APIs.

/ Version             |  TTI  (mobile)        | TTI (desktop) | 
|---------------------|---------------|---------------|
| [js-Date version](https://friendly-eats-demo-e71b7.web.app/js-date.html) | 2.1s | 0.6s |
| [java.time version](https://friendly-eats-demo-e71b7.web.app/java-time.html) | 2.2s | 0.7s |

 TTI (time-to-interactive) shown in the table above was taken from PageSpeed analysis. So yes, 
the java.time version is slower according to that analysis. Whether that amount is significant will depend on your use case.
Consider that JSJoda and React are fixed-size costs though. Being a small demo app, they 
 are disproportionately big. As application code grows over time with features their 
relative size will reduce ofc.

The memory usage for both apps was roughly the same, as observed in a recent version of Chrome.

What about download size? Obviously the java.time version will be draining the poor user's data allowance 
much faster? Well, let's imagine that every time a user visits these apps, the Clojurescript code has 
been changed and released, so cannot be retrieved from cache and must be re-downloaded.
It will only take 2-3 visits before the total amount of data downloaded across those visits is greater 
in the js/Date version. This is because the JS dependencies (JSJoda, React) are downloaded separately, 
and so are cacheable. This is very simple to set up and I would put it in the 'no brainer' category
if data allowance is a significant issue for an app. 

Of course real applications can go further and split the compiled application source into modules 
if that will help. Maybe we could modularize the js/Date version so Deja-Fu can be cached over visits, 
but my main point here is YMMV.

Are there more metrics that we should look at here? Please suggest anything you think is significant.

# And now, a twist

Did you notice any bugs?

The libraries have been weighed up against each other performance-wise, but
of course that is only one part of the story. 

Based on my impressions of the Clojure community, I think that the majority of Clojurescript developers
 also develop in Clojure, on the jvm. That means they will be using java.time (or if not yet,
will be doing so sooner or later). The Date/time thing is complicated, sometimes in
ways that are obvious to us, but crucially, sometimes not. What makes things needlessly more
difficult is having to use unfamiliar APIs. It's not just different method
signatures, but actual semantics. For example, if you 'add' a month to the 31st January, what happens?
A decision had to be made by the API authors, and that decision was made differently in java.time vs js/Date.

So I'll leave it to the reader to spot the bugs (I have deliberately put 2 in each version), but 
just offer a general word of warning that if you have
measured and found a worthwhile performance gain with some software you manage, make sure you consider 
the cost of that. Cost which 
might be measured in developer time and hard-to-spot bugs.

## Is this a fair test?

Any performance test needs to be taken with a large pinch of salt. The main take-away I would hope readers
get from this is that that JSJoda should not be dismissed out of hand for some common use cases, just 
as we don't generally dismiss Clojure because it is 'slower' or more 'memory hungry' compared to C. For many apps,
just adding a jsjoda dependency and maybe scanning the network tab in Chrome dev tools to see how it affects the
app overall will suffice.

All that being said, I have chosen some requirements for the app where I need to use to the much 
maligned js/Date API instead of Deja-Fu api, if not using java.time. 

Also, if the app needed to do custom parsing and formatting, for the java.time version I'd need to bring in a JSJoda addon, 
which takes TTI up to 2.7 seconds on mobile, whereas with the js/Date version TTI would be the same.

# If JSJoda is so great, why bother with Tempo?

[Tempo](https://github.com/henryw374/tempo) is a work-in-progress attempt to make a date-time API with the common parts of java.time
and the new Temporal platform API for Javascript (to be available in browsers in the near future).

The fact that Temporal is a platform API is the big reason of course.

My feeling is there is sufficiently large overlap between Java and Javascript's platform API's to make a useful 
library that will suit cross-platform 
library authors needing some basic date/time functionality, such as [Malli](https://github.com/metosin/malli/issues/49) and 
as perhaps also as a basis for a 'lite' version of the Tick library. 

# Conclusion

There may well be Clojurescript applications where JS-Joda would be inappropriate, but my feeling is 
for typical Clojurescript web applications it's not an issue.

Maybe I'm wrong though. I'd definitely be interested to hear about your opinions on this
and Clojurescript build sizes in general. What are you shipping? How did you make decisions 
about build was acceptable (including
the use of Clojurescript itself)?

Feel free to use [this thread on Clojureverse]() to comment. 

# Miscellaneous Thoughts on Deja-Fu - Chop this bit out?

Let's assume there are some applications where js-joda would not be acceptable. 

Deja-Fu's API offers a standalone `Time` type and LocalDate and LocalDateTime types, which wrap 
goog.date's Date and Datetime respectively, both of which wrap js/Date objects.

[Cljs-time]() is similar in that it wraps that same goog.date API, but doesn't include a Time entity.


* epochMs - you get different results for same date bc user offset. returns different results depending on my time zone.

Why would a date (year-month-day) have an epochMs getter? Same question of the other types. Leaky abstraction.

* mutable objs - but probably not an issue

* just cljs, not Clojure.

* no explicit clock. set `goog.now` when testing?

* data-literals. It would work to use time-literals server side and deja-fu literals in the client

* data-style API (destructure, assoc, update)

but...  objs here are not data?

e.g. cannot assoc anything I like

* ineviatably short-lived (Temporal)



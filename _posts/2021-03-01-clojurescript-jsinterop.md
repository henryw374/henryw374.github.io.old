---
layout: post
title: Accessing Javascript objects from Clojurescript  
description: Using dot-access vs goog.object vs something else
category: clojure 
---

There are choices as to how you access Javascript objects from Clojurescript.

Starting with an appeal to authority:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In concrete terms, sounds like<br>✓ (.-length &quot;abc&quot;)<br>X (.-length <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3})<br>✓ (goog.object/get <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3} &quot;length&quot;)</p>&mdash; Mike Fikes (@mfikes) <a href="https://twitter.com/mfikes/status/882585745424338944?ref_src=twsrc%5Etfw">July 5, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Firstly, I'll try to make a clear distinction between JS data vs API: A data object is 
any object you could round-trip through JSON/stringify => JSON/parse. An object that appears 
 to be a data object because it only contains properties (ie no methods), may not be a data object, because those
properties might be [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)s.
That definition should be good enough for the purposes of this ponst, let's ignore Object.prototype etc
for now. 

I find it interesting wrt to the `(.-length "foo")` item in Mike's list. 
It's clear in the case of the string `"foo"` that this is not a data object 
and btw `goog.object/get` will not work to access `length`, or any other property of a string.

Consider this example though: 

```clojure
(goog.object/get #js[] "length")
```

That returns `0`, so demonstrating it is possible to use the `goog.object` API to access API properties.

So, we have a stylistic choice between that and `(.-length #js[])`. Why choose either one? 

It's easy to see that the goog.object one will survive advanced compilation, whereas in some 
cases the dot-access would need a type hint, for example: `(.-length ^js foo)`. 

So +1 for the goog.obj approach so far.

In working with the API of some JS object, it's quite common to both access properties and call methods:

```clojure
(let [foo ^js (some-fn)
      bar-prop (.-bar foo)]
    (.methodFoo foo (inc bar-prop)))
```  

As I pointed out before, we could have accessed `bar-prop` with `goog.object/get`, but
this example is being consistent in using dot access only for the API of `foo`. We could also have accessed `methodFoo` with 
goog.object/get (and then invoked it), but it should be clear it would not read too well.

So, Following a rule 'dot access only for APIs' means the code makes a clear statement that it is
working with API, not data - regardless of whether it is just properties, or both methods and properties we need to 
use. This comes as the cost of having to remember to put type hints in. I've come to think
 type hints aren't so bad, because now [type hints are documented](https://code.thheller.com/blog/shadow-cljs/2017/11/06/improved-externs-inference.html)
 I think it's easy enough to understand you just need `^js` and where I need to put it.
 
Now that's cleared up, which [dot-access](https://cljs.github.io/api/syntax/dot) is preferred, `a.b.c` or `(.. a -b -c)` ...?  


 
                                                          
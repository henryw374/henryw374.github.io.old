---
layout: post
title: Accessing Javascript objects from Clojurescript  
description: Using dot-access vs goog.object vs something else
category: clojure 
---

advanced compilation etc etc

There are choices as to how you access a Javascript object's methods and properties from Clojurescript.

Starting with an appeal to authority:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In concrete terms, sounds like<br>✓ (.-length &quot;abc&quot;)<br>X (.-length <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3})<br>✓ (goog.object/get <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3} &quot;length&quot;)</p>&mdash; Mike Fikes (@mfikes) <a href="https://twitter.com/mfikes/status/882585745424338944?ref_src=twsrc%5Etfw">July 5, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So that's David and Mike's opinion, now I want to look into the tradeoffs.

Firstly, I'll try to make a clear distinction between JS data vs API: A data object is 
any object you could round-trip through JSON/stringify => JSON/parse. An object that may appear 
 to be a data object because it only contains properties (ie no methods), may not be a data object, because those
properties might be [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)s or setters.
If the object came via a JS library, it's most likely not a data object unless the library documentation
explicitly says so.

That definition should be good enough for the purposes of this post, let's ignore prototypes etc
for now. 

I find the `(.-length "foo")` item in Mike's list interesting. 
It's clear in the case of the string `"foo"` that this is not a data object 
and btw `goog.object/get` will not work to access `length`, or any other property of a string.

But consider this example though: 

```clojure
(goog.object/get #js[] "length")
```

That returns `0`, so demonstrating it is possible to use the `goog.object` API to access API properties.

So, we have what appears to be just a stylistic choice between that and `(.-length #js[])`. Why choose either one? 

It's easy to see that the goog.object one will survive advanced compilation, whereas in some 
cases the dot-access would need a type hint, for example: `(.-length ^js foo)`. 

So +1 for the goog.obj approach so far I guess.

In working with the API of some JS object, it's quite common to both access properties and call methods:

```clojure
(let [foo ^js (some-fn)
      bar-prop (.-bar foo)]
    (.methodFoo foo (inc bar-prop)))
```  

As I pointed out before, we could have accessed `bar-prop` with `goog.object/get`, but
this example is being consistent in using dot access only for the API of `foo`. We could also have accessed `methodFoo` with 
goog.object/get (and then invoked it), but I don't think it would be idiomatic to do so.

So, Following a rule 'dot access only for APIs' means the code makes a clear statement that it is
working with API, not data - regardless of whether it is just properties, or both methods and properties we need to 
use. This comes as the cost of having to remember to put type hints in. I've come to think
 type hints aren't so bad, because now [type hints are documented](https://code.thheller.com/blog/shadow-cljs/2017/11/06/improved-externs-inference.html)
 I think it's easy enough to understand you just need to add `^js` when you first see the js object in scope.
 
 Yet another approach would be to use a library such as [js-interop](https://github.com/applied-science/js-interop). Using that,
 you would write `(j/call foo :bar 1)` for the js equivalent `foo.bar(1)`. Type hints are not needed if you use that 
 library, so it's definitely an alternative to consider.
 
So, now that's all cleared up, which [dot-access](https://cljs.github.io/api/syntax/dot) is preferred, `a.b.c` or `(.. a -b -c)` ...?

a.b.c does have [an issue](https://clojure.atlassian.net/jira/software/c/projects/CLJS/issues/CLJS-3315) 

### Addendum

Thomas Heller pointed out to me that if Google Closure did become able to optimise regular JS libraries (things that are now foreign)
at some point in the future, then anything interop that is using strings to access properties ( `js-interop` lib, `goog.get` & etc) would then be broken,
so that's something else to consider

 
                                                          
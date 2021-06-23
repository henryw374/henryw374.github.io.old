---
layout: post
title: Accessing Javascript objects from Clojurescript  
description: Using dot-access vs goog.object vs something else
category: clojure 
---

When doing advanced compilation with Clojurescript, the compiler assumes it can change any names in your
program, except when it's told to leave certain names as-is. This only comes up as an issue when accessing
a Javascript object's methods and properties. There are choices as to how you refer to these from Clojurescript and that's
what I'm going to look into here.

IMHO Clojurescript is somewhat lacking when it comes to official documentation, hence this blog post, and the need to quote from twitter:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In concrete terms, sounds like<br>✓ (.-length &quot;abc&quot;)<br>X (.-length <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3})<br>✓ (goog.object/get <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3} &quot;length&quot;)</p>&mdash; Mike Fikes (@mfikes) <a href="https://twitter.com/mfikes/status/882585745424338944?ref_src=twsrc%5Etfw">July 5, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I'm going to look into the tradeoffs of what David and Mike say there.

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

It's easy to see that the goog.object one will survive advanced compilation (meaning it is compiled to `[].length` or equivalent), whereas in some 
cases the dot-access might need a type hint (as in `(.-length ^js foo)`) to avoid `length` being renamed.

So +1 for the goog.obj approach so far I guess.

In working with the API of some JS object, it's quite common to both access properties and call methods:

```clojure
(let [foo ^js (some-fn)
      bar-prop (.-bar foo)]
    (.methodFoo foo (inc bar-prop)))
```  

We could have accessed `bar-prop` with `goog.object/get`, but
this example is being consistent in using dot access only for the API of `foo`. We could also have accessed `methodFoo` with 
goog.object/get (and then invoked it), but I don't think it would be idiomatic to do so.

So, Following a rule 'dot access only for APIs' means the code makes a clear statement that it is
working with API, not data - regardless of whether it is methods or properties we need to 
use. This comes as the cost of having to remember to put type hints in. I've come to think
 type hints aren't so bad, because now [type hints are documented](https://code.thheller.com/blog/shadow-cljs/2017/11/06/improved-externs-inference.html)
 I think it's easy enough to understand you just need to add `^js` when you first see the js object in scope.
 
 Yet another approach would be to use a library such as [js-interop](https://github.com/applied-science/js-interop). Using that,
 you would write `(j/call foo :bar 1)` for the js equivalent `foo.bar(1)`. Type hints are not needed if you use that 
 library, so it's definitely an alternative to consider.

If working with JS data, ie not API, then an alternative to `goog.object` is [Cljs Bean](https://github.com/mfikes/cljs-bean)
 
So, now that's all cleared up, which [dot-access](https://cljs.github.io/api/syntax/dot) is preferred, `a.b.c` or `(.. a -b -c)` ...?

Dot access in the style of `a.b.c` does have [an issue](https://clojure.atlassian.net/jira/software/c/projects/CLJS/issues/CLJS-3315),
which is a shame (until fixed) because to me that seems pretty tasteful.

### Addendum

Thomas Heller pointed out to me that if Google Closure did become able to optimise regular JS libraries (things that are now foreign)
at some point in the future, then anything that is using strings to access JS APIs ( `js-interop` lib, `goog.get` & etc) would then be broken,
so that's something else to consider


 
                                                          
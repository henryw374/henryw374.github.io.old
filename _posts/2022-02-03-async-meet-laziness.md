---
layout: post
title: hey Async! meet Laziness
description: Combining Java's async with Clojure's laziness yields some interesting behaviour
category: clojure
---

Consider:

```clojure

(doseq [x (range 1000000)])

```

Since `range` returns a lazy sequence and `doseq` does not retain the head of the
sequence, there will only be one element of the sequence realized at every step of the `do`.1000000

Ok, let's split the creation and consumption of the lazy sequence in a chained promise. 
I am using [the promesa library](https://github.com/funcool/promesa) on the jvm, which
is a thin wrapper over java.util.concurrent#CompletableFuture.

```clojure

(-> (p/resolved (range 1000000))
    (p/then (fn [xs]
              (doseq [x xs]))))

```

It looks like it should be just as lazy. Nowhere is the code above retaining a reference to 
the head of the sequence - and yet it is! 

The `then` promise internally has a reference to the preceding promise and that promise has a reference
to its result - the head of the sequence. When the first promise returns the sequence is unrealized,
but as the subsequent promise consumes the sequence it is realized and the head retained 
by the preceding one!

What happens if there is a longer chain of promises? A promise executing in a chain
only has reference to the preceding one. The preceding one has lost its reference to the 
next one upstream, so in a chain just the current and immediately preceding one are not 
gc-able. 

So? Well imagine you are streaming results out of a db for example - that might be 
modelled as a lazy seq, which is consumed through e.g. doseq and written out to a 
stream. Sounds like a nice scalable solution, but if the db request results in a 
promise it might seem natural to keep chaining that result on. 

# Further thoughts 

Does this apply to 

* js promises? I would imagine so 
* core.async channels? I would guess not  
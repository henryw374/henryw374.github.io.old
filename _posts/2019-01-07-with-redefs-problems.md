---
layout: post
title: When to avoid with-redefs
---

(with-redefs)[https://clojuredocs.org/clojure.core/with-redefs] is a handy clojure.core function to use when you want to:

* temporarily redefine one or more vars
* AND you want these redefinitions to apply across thread boundaries

An alternative for doing something similar is (with-bindings)[https://clojuredocs.org/clojure.core/with-bindings], and that is slightly different in that
the new binding is only seen within the context of the current thread. If for example, you use with-bindings on a var and that var is called when doing a `clojure.core/map` operation then it is possible you won't see the temp binding, since `map` is lazy and may be executed in a different thread.

Also, `with-bindings` requires the rebound vars are dynamic. 

So... with-redefs is generally a better go-to tool than with-bindings?

Well, it is not without problems. Consider rebinding a function foo

```
(with-redefs [a.b.c/foo my-temp-fn]
  ... body that calls foo at some point, possibly from another thread)
```

Will the body here always see your temp binding?

** No **

Why not?

Well, interleaved threads is one situation. 

Let's say this code is called from two threads and this is the order of events:


Thread A executes the `with-redefs`
Thread B executes the `with-redefs`
Thread A executes the body and exits the `with-redefs` block (thereby restoring the root binding of foo)
Thread B now executes the body and does not see the temp binding!!

The docs for `with-redefs` I've seen say this is handy for tests, which kind of implies that you'd use `alter-var-root` or something in non-test code,  but there's nothing to stop you hitting an interleaving problem in tests either.

Conclusion:

Use `alter-var-root` cautiously. Remember this blog post when scratching your head about why some code is not seeing a temp binding.


---
layout: post
title: Tick library on Shadow-cljs 'Just Works'
description: Small but significant releases for npm and babashka users
category: clojure 
---

[tick](https://github.com/juxt/tick) is a Clojure(Script) library for working with time and is built atop the `java.time` api (and uses a pure-JS impl of java.time on javascript platforms).

New in release 0.4.25-alpha is that it 'just works' on Shadow-cljs:

```
echo '{:deps {org.clojure/clojurescript {:mvn/version "1.10.741" } tick {:mvn/version "0.4.25-alpha"} thheller/shadow-cljs {:mvn/version "2.9.8"} }}' > deps.edn
echo '{:deps {}}' > shadow-cljs.edn
echo '{}' > package.json

npm install shadow-cljs --save-dev && npx shadow-cljs browser-repl
```

In the repl:

```
cljs.user=> (require '[tick.alpha.api :as t])
nil
cljs.user=> (t/today)
#time/date "2020-05-31"

```

Why is this news? Well, previously Shadow and other npm-using Clojurescript users had to include an extra shim library and manually include the js-joda dependencies in their package.json. A desire to move to the latest (and now scope-named) '@js-joda/core' packages, [a fix in the latest Clojurescript release](https://clojure.atlassian.net/browse/CLJS-3138) and improvement in my understanding of [how to package Clojurescript libraries](http://widdindustries.com/cljs-npm-libraries/) has resulted in  the latest release making things easy for more users. Note, there are still a few [need-to-knows for Tick Clojurescript users](https://juxt.pro/tick/docs/index.html#_clojurescript), but not too many!

# Babashka

In other news [cljc.java-time](https://github.com/henryw374/cljc.java-time) (the date-time library Tick depends on) now works on a third platform [babashka](https://github.com/borkdude/babashka/)!

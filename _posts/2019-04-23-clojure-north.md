---
layout: slide
title: Cross-Platform Date-Time Awesomeness
description: My talk from Clojure/North 2019
transition: slide
---

<section data-markdown>
## Cross Platform Date-Time Awesomeness



#### by Henry Widd

#### @henryw374


</section>

<section data-markdown=""  data-separator-notes="^Note:">
```clojure
(ns foo.cljc
  (:require
    #?(:clj  [clj-time.core  :as ct]
       :cljs [cljs-time.core :as ct])))

(ct/before? (ct/today) (ct/today))
; => true (in cljs)

```
</section>


<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
  #### Why Cross-Platform Code?

  ![alt text](/images/cljc.jpg)
  

</section>


<section data-markdown=""  data-separator-notes="^Note:">
    
  ![alt text](/images/plumbing-brain.svg)
  

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
![alt text](/images/plumbing-brain-cljc.svg)
</section>

<section data-separator-notes="^Note:">
  <code class="language-clojure hljs">#?@(<span class="hljs-name">...</span>)</code>
  <p class="fragment">Maths: kixi.stats</p>
  <p class="fragment">Strings: cuerdas</p>
  <p class="fragment">Data: frankiesardo/linked</p>
  <p class="fragment">Dates: ____ </p>
</section>  


<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
#### java.time & js-joda date objects
![jsr310 domain](/images/jsr310-domain.svg  "jsr310-domain" ) 
</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
### Cross-Platform java.time

```clojure
(ns foo
  (:require
    #?(:cljs [java.time :refer [LocalDate]])   ;(1)
    [time-literals.read-write])
   #?(:clj (:import [java.time LocalDate])))   ;(2)
   
  (def d (. LocalDate parse "2015-10-21"))     ;(3)
  ;=> #time/date"2015-10-21"                   ;(4)
  
  (= d (. LocalDate now))                      
  ; => false

  (-> d                       
      (.atStartOfDay (. ZoneId systemDefault))) ;(5)
; #time/zoned-date-time"2015-10-21T00:00-04:00[America/New_York]"
  
```

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## A Clojure wrapper for x
* Provides?
  *  Avoid Interop Code
  *  Clojurey Version of the API (e.g. data in/out)
  *  Extra features (API, Immutability etc)

* Need to know:
  * Do I need to know x? Is the API tied to x? 
  * Do x's docs, community & Stack overflow translate

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## cljc.java-time

 ```clj
(ns my.cljc
  (:require [cljc.java-time.local-date :as local-date]) ;(1)
      
(def d (local-date/parse "2019-01-01"))                 ;(2)

(local-date/at-start-of-day d)
; => #time/date-time"2020-12-01T00:00"   
 ```

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## clojure.java-time

* Clojurey-ness
  * Data API - fields and units
  * Single namespace

* Features
  * `(with-clock ...)`
  * Conversions to/from Joda, java.sql.* & other date types

</section>

<section data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
<h2>Tick</h2>
<br/>
<p class="fragment">Authored by Malcolm Sparks @ JUXT</p>
<p class="fragment">Power Tools for working with Time</p>


</section>  

<section  data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
How much annual leave has an employee used up?
<br/>
<p class="fragment">Inputs:</p>
<br/>
<p class="fragment">what periods have they been out of office?</p>
<p class="fragment">Relevant public holiday calendar</p>
<p class="fragment">annual leave allowance start/end</p>
 
</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:" style="">
## Interval Operations
  ```clojure
(require '[tick.alpha.api :as t])

(def working-days
  (t/difference                                 ;(1)
    #:tick{:beginning #time/date "2012-01-01", 
           :end       #time/date "2020-01-01"}
   weekends 
   public-holidays))
                    
; => (#:tick{:beginning #time/date "2012-01-03" ;(2) 
;            :end       #time/date "2012-01-07"} 
;     .... )

(def holidays-by-year
  (->
    (t/intersection
      working-days
      employee-holidays)
    (t/group-by t/year)))                       ;(3)
    
; => {#time/year"2012" (#:tick{:beginning #time/date "2012-01-03", :end #time/date "2012-01-07"} .... )
;    {#time/year"2013" (#:tick{:beginning #time/date "2013-04-05", :end #time/date "2013-04-13"} .... )
;    ...}

```

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## Tick

```clojure 
(-> (t/time "10am") 
    (t/on "2019-04-20") 
    (t/in "America/Toronto"))
```

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
### Recap

```clojure
 [tick] ;(alpha)
   [cljc.java-time]
     [cljs.java-time]
       [cljsjs/js-joda]
   
 [time-literals]
   [cljs.java-time]
```
</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## For The Future ...
* tc39/proposal-temporal & Intl api
* specs (and generators, similar to `s/inst-in`)
* tick - '1.0' - more docs etc
* automated tracking and publishing of discrepancies between java.time and js-joda
   * incl. methods added to java.time in Java 9 and higher
   * decide what cljc.java-time should do

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
## To Wrap up...
* I got the cross platform date-time APIs I wanted
  * big shoutout to the JS-Joda authors & others
* Consider defaulting to cljc

</section>

<section data-markdown="" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:">
  ## Links

  @henryw374

* [My Blog](http://widdindustries.com/)
* [tick](https://github.com/juxt/tick)
* [time-literals](https://github.com/henryw374/time-literals)
* [cljc.java-time](https://github.com/henryw374/cljc.java-time)
</section>

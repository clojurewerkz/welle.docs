---
title: "With Clojure and Riak: integration with other libraries"
layout: article
---

## About this guide

This guide covers:

 * Why integrate with other libraries?
 * Using welle.ring.store: [Ring session store](https://github.com/mmcgrana/ring/blob/master/ring-core/src/ring/middleware/session/store.clj) implementation on top of Riak
 * Using welle.cache: core.cache implementation on top of Riak

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What Version of Welle does this guide cover?

This guide covers Welle 3.0, including development releases.


## Introduction

Welle is a "batteries included" library: it provides integration
points with popular Clojure libraries. For example, a fairly common
use case for Riak is session store for Web applications and HTTP
services. Another one is cache store, especially when data volume
often exceeds total amount of RAM, caching is done for a longer period
of time (days, weeks or more) or computation of cached values is very
expensive so replication and availability of the cache become a
concern.

Welle supports these use cases by providing implementations of session
store and cache store protocols found in
[Ring](https://github.com/mmcgrana/ring/blob/master/ring-core/src/ring/middleware/session/store.clj)
and [core.cache](https://github.com/clojure/core.cache).


## Ring Session Store On Riak

To use Riak Ring session store, require `clojurewerkz.welle.ring.session-store` and use `clojurewerkz.welle.ring.session-store/welle-store` function like so:

``` clojure
(ns my.service
  (:require [clojurewerkz.welle.core :as wc]
            [clojurewerkz.welle.ring.session-store :refer :all]))

(let [conn  (wc/connect)
      store (welle-store conn "web_sessions")]
  ...)
```

It is possible to pass `:r`, `:w` and `:content-type` options that
will be used for reads and writes:

``` clojure
(ns my.service
  (:require [clojurewerkz.welle.core :as wc]
            [clojurewerkz.welle.ring.session-store :refer :all]))

(let [conn  (wc/connect)
      store (welle-store conn "web_sessions"
                             ;; r
                             3
                             ;; w
                             3
                             "application/json")]
  ...)
```

By default, `:w` and `:r` of 2 will be used and `:content-type` is
`com.basho.riak.client.http.util.Constants/CTYPE_JSON_UTF8`


## core.cache Implementation Backed By Riak

core.cache implementation comes from the `clojurewerkz.welle.cache`
namespace. To create cache store instances, use the
`basic-welle-cache-factory` function:

``` clojure
;; empty cache, default bucket name ("cache_entries"), default serialization (JSON) and :w value
(basic-welle-cache-factory conn)

;; empty cache, default serialization (JSON), custom bucket name
(basic-welle-cache-factory conn "cache")

;; cache with seed entries, custom serialization format, custom :w value
(basic-welle-cache-factory conn "cache_entries" {"a" 1 "b" "cache" "c" 3/4} "application/clojure" 2)
```

It is common to first set bucket properties as described in the
[working with Riak
buckets](http://localhost:4000/articles/buckets.html) guide and then
pass a name of that bucket to the cache store factory function.


## data.json extensions for dates

Welle uses [Cheshire](https://github.com/dakrone/cheshire) for JSON
serialization. One nice property of that library is that it is is
extensible, thanks to Clojure protocols it is built on. While JSON has
very good set of fundamental data types, it does not define
serialization format for dates.  Because serialization of dates is
common (for example, for the Ring session store Welle provides), Welle
also has extensions for `clojure.data.json` that make it possible to
transparently serialize JodaTime dates. All you need to do is to
require one namespace:

``` clojure
(ns my.service
  (:require clojurewerkz.support.json))
```

Please note that this extension assumes you use
[clj-time](https://github.com/seancorfield/clj-time) to work with
dates, although `java.util.Date` instances are also supported by
converting them to UTC JodaTime date/time instances.



## What To Read Next

This is the last guide. Congratulations! You may want to take a look
at [the guide list](/) or follow
[@ClojureWerkz](http://twitter.com/ClojureWerkz) on Twitter where we
announce important changes to libraries and documentation.



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

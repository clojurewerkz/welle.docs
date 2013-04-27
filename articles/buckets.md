---
title: "Welle, a Riak Clojure client: Working with Riak buckets"
layout: article
---

## About this guide

This guide covers:

 * What are Riak buckets
 * How to set/update bucket properties with Welle
 * How to list buckets
 * Bucket properties, what do they mean
 * How to fetch bucket properties
 * How to update bucket properties

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 1.5.


## Introduction

Riak stores data in *buckets*. They are similar in purpose to tables in relational databases and collections in MongoDB. Buckets have *properties* that
determine how data in a bucket is replicated, how conflicts are resolved, quorum defaults for read and write operations and so on.

Applications may choose to use default bucket properties or set them before storing data in it. Properties can be changed at any time
and some of them are just defaults for read and write operations on keys (so, they can be specified for individual operations, too).

Lets take a look at how bucket properties are set (updated).


## How To Set And Update Bucket Properties

To update bucket properties, you use `clojurewerkz.welle.buckets/update` function:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]))

(wc/connect!)

;; set bucket properties to (effectively) ignore conflicts
;; and replicate objects in this bucket 5 times in the cluster
(wb/update "a.bucket" :last-write-wins true :n 5)
```

When invoked without any options (just bucket name), it will use all defaults. Bucket properties can be updated multiple times:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]))

(wc/connect!)

;; set bucket properties
(wb/update "a.bucket" :last-write-wins true)

;; update them again
(wb/update "a.bucket" :last-write-wins false)
```

### Riak Bucket Properties

Welle supports setting the following [bucket properties](http://wiki.basho.com/HTTP-Set-Bucket-Properties.html):

 * `:last-write-wins` (true or false): whether to ignore object history (vector clock) when writing. Buckets that are used for caching often set this to true.
 * `:allow-siblings` (true or false): whether to allow sibling objects to be created (concurrent updates)
 * `:n-val`: the number of replicas for objects in this bucket
 * `:r`, `:w`, `:dw`, `:rw`, `:pr`, `:pw` (integer or one of the `com.basho.riak.client.cap.Quora` values): default quorum values for operations on keys in the bucket
 * `:basic-quorum`: should a read request return early in some failure cases? For example, when a quorum of nodes respond with "not found".
 * `:enable-search` (true or false): whether Riak Search is enabled for the bucket
 * `:backend`: when using [riak_kv_multi_backend](http://wiki.basho.com/Storage-Backends.html), which named backend to use for the bucket
 * `:not-found-ok`: should a vnode returning notfound for a key increment the r tally? Use false for higher consistency, true for higher availability.
 * `:small-vclock`, `:big-vclock`, `:young-vclock`, `:old-vclock`: used for vector clock pruning (to [control vector clock growth](http://wiki.basho.com/HTTP-Get-Bucket-Properties.html))


Note that some properties may depend on your Riak cluster configuration, transport used (HTTP vs Protocol Buffers) and Riak version. 


## How to list buckets

Use `clojurewerkz.welle.buckets/update` function. It takes no arguments and returns a set of strings (bucket names in the cluster):

{% gist ddb629aa25c9e1c873bd %}



## How To Fetch Bucket Properties

Use `clojurewerkz.welle.buckets/fetch` function. It takes bucket name as the only argument and returns bucket properties as a Clojure map:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]))

(wc/connect!)

;; set bucket properties
(wb/update "a.bucket" :last-write-wins true :n 5)

;; fetch them back as a Clojure map
(wb/fetch "a.bucket")
; => {:r #<Quorum com.basho.riak.client.cap.Quorum@333b68cb>, :not-found-ok true, :young-vclock 0,
; => :backend nil, :n-val 5, :search false, :last-write-wins true, :small-vclock 0, :basic-quorum false,
; => :dw #<Quorum com.basho.riak.client.cap.Quorum@5e9df8>, :rw #<Quorum com.basho.riak.client.cap.Quorum@5e9df8>,
; => :big-vclock 0, :name welle.test.a-bucket, :old-vclock 0, :pr #<Quorum com.basho.riak.client.cap.Quorum@333b68ac>,
; => :pw #<Quorum com.basho.riak.client.cap.Quorum@333b68ac>, :w #<Quorum com.basho.riak.client.cap.Quorum@5e9df8>, :allow-siblings false}
```


## What To Read Next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Key/Value Operations](/articles/kv.html)
 * [Secondary indexes](/articles/2i.html)
 * [Links and link walking](/articles/links.html)
 * [Map/Reduce](/articles/mapreduce.html)
 * [Integration with 3rd party libraries](/articles/integration.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on
Twitter or the [Welle mailing
list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you
do not like the guide style or grammar or discover spelling
mistakes. Reader feedback is key to making the documentation better.

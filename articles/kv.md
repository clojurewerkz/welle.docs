---
title: "Welle, a Riak Clojure client: Storing and fetching data in Riak with Clojure"
layout: article
---

## About this guide

This guide covers:

 * How to store data in Riak with Welle
 * How to fetch data
 * How to delete data

This work is licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by/3.0/">Creative Commons
Attribution 3.0 Unported License</a> (including images &
stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 2.0, include development releases.


## Storing values

Riak has several components to it but it is a key/value store at
heart. You interact with it using functions in the
`clojurewerkz.welle.kv` namespace. For example,
`clojurewerkz.welle.kv/store` function stores objects in Riak.  It
takes a connection, bucket name, key, and value:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv]))

(let [conn   (wc/connect)
      bucket "things"]
  (wb/create conn bucket)
  (kv/store  conn bucket "a-key" (.getBytes "value")))
```


### Automatic Serialization (for Common Formats)

In the example above we store some data as bytes: Riak is content
agnostic and will happily store whatever you throw at it.  However,
most applications work with data more structured than raw bytes, for
example, as JSON documents or text in the CSV format.  To make
developer experience a bit nicer, Riak will store content type of the
stored value and Welle will automatically serialize stored value if
content type is one of

 * JSON
 * JSON in UTF-8
 * Clojure data (that can be read by the Clojure reader)
 * Text
 * Text in UTF-8

Content type is passed as one of the optional arguments to the same
function, `clojurewerkz.welle.kv/store`:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv]))

(let [conn   (wc/connect)
      bucket "accounts"
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key}]
  (wb/create conn bucket)
  (kv/store  conn bucket key val {:content-type "application/json; charset=UTF-8"}))
```

It is common to use constants provided by the Riak Java client (that
Welle uses under the hood):

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import com.basho.riak.client.http.util.Constants))

(let [conn   (wc/connect conn)
      bucket "accounts"
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key}]
  (wb/create conn bucket)
  (kv/store  conn bucket key val {:content-type Constants/CTYPE_JSON_UTF8}))
```

However, Riak Java client constants do not include Clojure data
content type. To use it, pass "application/clojure" as the content
type:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv]))

(let [conn   (wc/connect)
      bucket "accounts
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key :created-at (java.util.Date.) :hacks #{"clojure" "java" "ruby" "scala" "erlang"}}]
  (wb/create conn bucket)
  ;; stores data serialized to be read by the Clojure reader, so the set and the date will be
  ;; transparently read back when this value is fetched.
  (kv/store conn bucket key val {:content-type "application/clojure"}))
```

Serialized values will be deserialized automatically by Welle when you
fetch them, as you will see later in this guide.



### Skipping Automatic Deserialization

`clojurewerkz.welle.kv/fetch` supports a new boolean option
`:skip-deserialize` that allows automatic deserialization to be
skipped.


### Main Consistency/Availability Options

Riak lets you trade some availability for consistency (or vice versa) for individual requests. To do so, use the following
options `clojurewerkz.welle.kv/store` accepts:

 * `:w`: write quorum, how many replicas to write to before returning a successful response
 * `:dw`: durable write quorum, how many replicas to commit to durable storage before returning a successful response
 * `:pw`: primary write quorum, how many replicas to commit to primary nodes before returning a successful response
 * `:resolver`: a function that takes a collection of Riak object maps and returns a single one by applying a conflict resolution logic
 * `:retrier`: a function that takes a callable and implements a retries logic. Usually the default retrier is good enough.

Default values for these options are [defined at the bucket level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs),
`com.basho.riak.client.cap.Quora` or
`com.basho.riak.client.cap.Quorum` instances.

### User Metadata

Riak lets you associate some *metadata* with each value
stored. Metadata is a map and is passed to
`clojurewerkz.welle.kv/store` using the `:metadata` option:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import java.util.UUID))

(let [conn   (wc/connect)
      bucket "blobs"
      key    (str (UUID/randomUUID))
      val    (slurp "/tmp/raw/image000001.raw")]
  (wb/create conn bucket)
  ;; stores data with a piece of metadata associated with it
  (kv/store conn bucket key val {:metadata {"format" "raw"}}))
```

A common example of metadata is audio, image and video files
metadata. If you store file contents in Riak, it's natural to store
file metadata information as user metadata. Other examples include
document authorship and copyright information, publication date, ISBNs
and so on.

Metadata will be transparently turned into a Clojure map when stored
value is fetched, as you will see later in this guide.


### Other Options

 * `:last-modified`: last value modification date
 * `:return-body`: should the store operation return the new data item and its metadata?
 * `:if-none-match`: only store if bucket/key does not exist
 * `:if-not-modified`: only store if the vclock supplied on store matches the vclock in Riak





## Fetching Values

To fetch a stored value, use `clojurewerkz.welle.kv/fetch`
function. In the simplest case it takes a connection, bucket name, and
key:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import com.basho.riak.client.http.util.Constants))

(let [conn   (wc/connect)
      bucket "accounts"
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key}]
  (wb/create conn bucket)
  ;; stores data serialized as JSON
  (kv/store bucket key val :content-type Constants/CTYPE_JSON_UTF8)
  ;; returns a Riak response
  (kv/fetch bucket key))
```

It returns *an immutable response map* with the following keys:

 * `:result`: one or more objects returned by Riak
 * `:vclock`: vector clock of the response
 * `:has-siblings?`: true if response has siblings
 * `:has-value?`: true if response is non-empty
 * `:modified?`: false when conditional GET returned a non-modified response
 * `:deleted?`: true if this object has been deleted but there is a vclock for it

To obtain a previously stored result, take `:result` from the
response. Note that for `clojurewerkz.welle.kv/fetch` `:result` will
contain a **list of values**.  In an eventually consistent system like
Riak it is sometimes possible that multiple versions of an object will
be stored in a cluster. They are called
[siblings](http://docs.basho.com/riak/latest/theory/concepts/Vector-Clocks/#Siblings). We
won't get into siblings and conflicts resolution in this guide, just
be aware of this possibility and that `clojurewerkz.welle.kv/fetch`
returns a response where `:result` is a list.

Granted, most of the time (in systems that are light on writes, almost
always) the list will only contain one value. Fortunately, Clojure has
us covered here: it is possible use positional destructuring to get
the value without additional calls to `clojure.core/first`:

``` clojure
;; fetch an object (possibly with siblings)
;; here we use positional destructuring to keep the code concise and idiomatic
(let [{:keys [result] (kv/fetch conn bucket key)}
      [val]           result]
  val)
```

So if you are sure that there will be no conflicts, this practice is
encouraged. Alternatively, you can use
`clojurewerkz.welle.kv/fetch-one` to automatically get only the first
value from the `:result`.

So far we haven't passed any arguments to
`clojurewerkz.welle.kv/fetch`. In case you need to do it, it is very
similar to how you do it when storing data:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import com.basho.riak.client.http.util.Constants))

(let [conn   (wc/connect)
      bucket "accounts"
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key}]
  ;; replicate values 3 times in the cluster
  (wb/create conn bucket {:n-val 3})
  ;; stores data serialized as JSON to 2 vnodes
  (kv/store conn bucket key val {:content-type Constants/CTYPE_JSON_UTF8 :w 2})
  ;; fetches it back, 2 vnodes must respond
  (kv/fetch conn bucket key {:r 2}))
```

### Main Consistency/Availability Options

Riak lets you trade some availability for consistency (or vice versa)
for individual requests. To do so, use the following options
`clojurewerkz.welle.kv/store` accepts:

 * `:r`: the read quorum, how many replicas need to agree when retrieving the object
 * `:pr`: the primary read quorum, how many primary replicas need to be online when doing the read

Default values for these options are [defined at the bucket
level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs),
`com.basho.riak.client.cap.Quora` or
`com.basho.riak.client.cap.Quorum` instances:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import com.basho.riak.client.http.util.Constants
           com.basho.riak.client.cap.Quora))

(let [conn   (wc/connect)
      bucket "accounts"
      key    "novemberain"
      val    {:name "Michael" :age 27 :username key}]
  ;; replicate values 3 times in the cluster
  (wb/create conn bucket {:n-val 3})
  ;; stores data serialized as JSON to 2 vnodes
  (kv/store conn bucket key val {:content-type Constants/CTYPE_JSON_UTF8 :w 2})
  ;; fetches it back, all vnodes must respond
  (kv/fetch conn bucket key {:r Quora/ALL}))
```

### Other Options

 * `:resolver`: a function that takes a collection of Riak object maps and returns a single one by applying a conflict resolution logic
 * `:retrier`: a function that takes a callable and implements a retries logic. Usually the default retrier is good enough
 * `:basic-quorum` (true or false): whether to return early in some failure cases (eg. when `:r` is 1 and you get 2 errors and a success `:basic-quorum` set to true would return an error)
 * `:notfound-ok` (true or false): whether to treat notfounds as successful reads for the purposes of `:r`
 * `:vtag`: when accessing an object with siblings, [which sibling to retrieve](http://docs.basho.com/riak/latest/dev/references/http/fetch-object/#Siblings-examples)
 * `:if-none-match` (date): a date for [conditional get](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). Only supported by HTTP transport.
 * `:if-modified-vclock`: a vclock instance to use for conditional get. Only supported by Protocol Buffers transport.
 * `:return-deleted-vlock` (true or false): should tombstones (objects that have been deleted but not yet resolved/GCed) be returned?
 * `:head-only` (true or false): should the response only return object metadata, not its value?

Default values for these options are also [defined at the bucket level](/articles/buckets.html).

### Tombstones Handling

In eventually consistent systems such as Riak, deleted objects may
sometimes "reappear" due to concurrent modifications.  Welle by
default will filter out tombstones in
`clojurewerkz.welle.kv/fetch`. If you want to retrieve all objects
including tombstones, pass `:return-deleted-vlock` as `true` to
`clojurewerkz.welle.kv/fetch`. This behavior was added in Welle
`1.4.0`.


## Deleting values

To delete a value with Welle, you use `clojurewerkz.welle.kv/delete`
that takes a connection, bucket name, and key to delete:

``` clojure
(ns welle.docs.examples
  (:require [clojurewerkz.welle.core    :as wc]
            [clojurewerkz.welle.buckets :as wb]
            [clojurewerkz.welle.kv      :as kv])
  (:import java.util.UUID))

(let [conn   (wc/connect)
      bucket "blobs"
      key    (str (UUID/randomUUID))
      val    (slurp "/tmp/raw/image000001.raw")]
  (wb/create conn bucket)
  (kv/store  conn bucket key val {:metadata {"format" "raw"}})
  (kv/delete conn bucket key))
```

### Main Consistency/Availability Options

 * `:r`: read quorum, how many replicas need to agree when retrieving the object
 * `:pr`: primary read quorum, works like `:r` but requires that the nodes read from are not fallback nodes
 * `:w`: write quorum, how many replicas to write to before returning a successful response
 * `:dw`: durable write quorum, how many replicas to commit to durable storage before returning a successful response
 * `:pw`: primary write quorum, how many replicas to commit to primary nodes before returning a successful response
 * `:rw`: quorum for both operations (get and put) involved in deleting an object

Default values for these options are [defined at the bucket level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs),
`com.basho.riak.client.cap.Quora` or
`com.basho.riak.client.cap.Quorum` instances.

### Other Options

 * `:vclock`: vector clocks to use for the update



### clojurewerkz.welle.kv/modify

`clojurewerkz.welle.kv/modify` is a new function that combines
`clojurewerkz.welle.kv/fetch` and `clojurewerkz.welle.kv/store` with a
user-provided mutation functions. The mutation function should take a
single Riak object as an immutable map and return a modified one.

In case of siblings, a resolver should be used.

`clojurewerkz.welle.kv/modify` will update modification timestamp of the object.

`clojurewerkz.welle.kv/modify` takes the same options as `clojurewerkz.welle.kv/fetch` and
`clojurewerkz.welle.kv/store`


### Conflcit Resolvers

`clojurewerkz.welle.kv/fetch`, and `clojurewerkz.welle.kv/store` now
accept a new option: `:resolver`. Resolvers are basically pure
functions that take a collection of siblings and return a collection
of Riak object maps.

Resolvers can be created using
`clojurewerkz.welle.conversion/resolver-from` which takes a function
that accepts a collection of deserialized (unless `fetch` was told
otherwise) values and applies any conflict resolution logic necessary,
returning a single object:

``` clojure
(require '[clojurewerkz.welle.conversion  :as cnv])

;; a resolver that always picks the first value
(cnv/resolver-from
  (fn [siblings]
    (first siblings)))
```

`clojurewerkz.welle.kv/fetch-one` now also supports resolvers via the
`:resolver` option.  It will raise an exception if siblings are
detected and no resolver is provided.



### Retriers

`clojurewerkz.welle.kv/fetch`, `clojurewerkz.welle.kv/fetch-one`,
`clojurewerkz.welle.kv/store`, `clojurewerkz.welle.kv/delete`, and
`clojurewerkz.welle.kv/index-query` now retry operations that fail due
to a network issue or any other exception.

By default, the operations will be retrier 3 times. It is possible to
provide a custom retrier using the `:retrier` option. Retriers can be
created using `clojurewerkz.welle.conversion/retrier-from` which takes
a function that accepts a callable (an operation that may need to be
retried) and needs to invoke it, handling exceptions and applying any
retrying logic needed.


`clojurewerkz.welle.conversion/counting-retrier` produces a retrier
that will retry an operation given number of times. This is the kind
of retrier Welle uses by default.


## What to read next

 * [Secondary indexes](/articles/2i.html)
 * [Links and link walking](/articles/links.html)
 * [Map/Reduce](/articles/mapreduce.html)
 * [Integration with 3rd party libraries](/articles/integration.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

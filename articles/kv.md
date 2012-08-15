---
title: "Welle, a Riak Clojure client: Storing and fetching data in Riak with Clojure"
layout: article
---

## About this guide

This guide covers:

 * How to store data in Riak with Welle
 * How to fetch data
 * How to delete data

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 1.2.


## Storing values

Riak has several components to it but it is a key/value store at heart. You interact with it using functions in
the `clojurewerkz.welle.kv` namespace. For example, `clojurewerkz.welle.kv/store` function stores objects in Riak.
It takes a bucket name, a key and a value:

{% gist 8336e8887bb37e258621 %}


### Automatic serialization (for common formats)

In the example above we store some data as bytes: Riak is content agnostic and will happily store whatever you throw at it.
However, most applications work with data more structured than raw bytes, for example, as JSON documents or text in the CSV format.
To make developer experience a bit nicer, Riak will store content type of the stored value and Welle will automatically serialize
stored value if content type is one of

 * JSON
 * JSON in UTF-8
 * Clojure data (that can be read by the Clojure reader)
 * Text
 * Text in UTF-8

Content type is passed as one of the optional arguments to the same function, `clojurewerkz.welle.kv/store`:

{% gist 62b9de5d322690b7efbf %}

It is common to use constants provided by the Riak Java client (that Welle uses underneath):

{% gist 4d67f00b740e1fc74724 %}

However, Riak Java client constants do not include Clojure data content type. To use it, pass "application/clojure" as the content type:

{% gist 65b3f1dcdcf7ee026cf8 %}

Serialized values will be deserialized automatically by Welle when you fetch them, as you will see later in this guide.


### Main consistency/availability options

Riak lets you trade some availability for consistency (or vice versa) for individual requests. To do so, use the following
options `clojurewerkz.welle.kv/store` accepts:

 * `:w`: write quorum, how many replicas to write to before returning a successful response
 * `:dw`: durable write quorum, how many replicas to commit to durable storage before returning a successful response
 * `:pw`: primary write quorum, how many replicas to commit to primary nodes before returning a successful response

Default values for these options are [defined at the bucket level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs), `com.basho.riak.client.cap.Quora` or `com.basho.riak.client.cap.Quorum`
instances.

### User Metadata

Riak lets you associate some *metadata* with each value stored. Metadata is a map and is passed to `clojurewerkz.welle.kv/store` using
the `:metadata` option:

{% gist 0b46d804809388ceb0f3 %}

A common example of metadata is audio, image and video files metadata. If you store file contents in Riak, it's natural to store
file metadata information as user metadata. Other examples include document authorship and copyright information, publication date,
ISBNs and so on.

Metadata will be transparently turned into a Clojure map when stored value is fetched, as you will see later in this guide.


### Other options

 * `:last-modified`: last value modification date
 * `:return-body`: should the store operation return the new data item and its metadata?
 * `:if-none-match`: only store if bucket/key does not exist
 * `:if-none-modified`: only store if the vclock supplied on store matches the vclock in Riak





## Fetching values

To fetch a stored value, use `clojurewerkz.welle.kv/fetch` function. In the simplest case it takes a bucket name and a key:

{% gist 9978c3d0ad6c4d668885 %}

It returns *a list of values* because in an eventually consistent system like Riak it is sometimes possible that multiple
versions of an object will be stored in a cluster. They are called [siblings](http://wiki.basho.com/Vector-Clocks.html#Siblings). We won't
get into siblings and conflicts resolution in this guide, just be aware of this possibility and that `clojurewerkz.welle.kv/fetch` returns
a list.

Granted, most of the time (in systems that are light on writes, almost always) the list will only contain one value. Fortunately,
Clojure has us covered here: it is possible use positional destructuring to get the value without additional calls to `clojure.core/first`:

{% gist af3dd517ea8f094a1a47 %}

So if you are sure that there will be no conflicts, this practice is encouraged.

So far we haven't passed any arguments to `clojurewerkz.welle.kv/fetch`. In case you need to do it, it is very similar to how you do
it when storing data:

{% gist 947cda5651405cffbda6 %}

### Main consistency/availability options

Riak lets you trade some availability for consistency (or vice versa) for individual requests. To do so, use the following
options `clojurewerkz.welle.kv/store` accepts:

 * `:r`: the read quorum, how many replicas need to agree when retrieving the object
 * `:pr`: the primary read quorum, how many primary replicas need to be online when doing the read

Default values for these options are [defined at the bucket level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs), `com.basho.riak.client.cap.Quora` or `com.basho.riak.client.cap.Quorum`
instances:

{% gist 41157189e87b56d46aa8 %}


### Other options

 * `:basic-quorum` (true or false): whether to return early in some failure cases (eg. when `:r` is 1 and you get 2 errors and a success `:basic-quorum` set to true would return an error)
 * `:notfound-ok` (true or false): whether to treat notfounds as successful reads for the purposes of `:r`
 * `:vtag`: when accessing an object with siblings, [which sibling to retrieve](http://wiki.basho.com/HTTP-Fetch-Object.html#Manually-requesting-siblings)
 * `:if-none-match` (date): a date for [conditional get](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). Only supported by HTTP transport.
 * `:if-modified-vclock`: a vclock instance to use for conditional get. Only supported by Protocol Buffers transport.
 * `:return-deleted-vlock` (true or false): if an object has been deleted, should the tombstone vclock be returned?
 * `:head-only` (true or false): should the response only return object metadata, not its value?

Default values for these options are also [defined at the bucket level](/articles/buckets.html).


## Deleting values

To delete a value with Welle, you use `clojurewerkz.welle.kv/delete` that takes a bucket name and a key to delete:

{% gist fc1c155149f8c12452a5 %}


### Main consistency/availability options

 * `:r`: read quorum, how many replicas need to agree when retrieving the object
 * `:pr`: primary read quorum, works like `:r` but requires that the nodes read from are not fallback nodes
 * `:w`: write quorum, how many replicas to write to before returning a successful response
 * `:dw`: durable write quorum, how many replicas to commit to durable storage before returning a successful response
 * `:pw`: primary write quorum, how many replicas to commit to primary nodes before returning a successful response
 * `:rw`: quorum for both operations (get and put) involved in deleting an object

Default values for these options are [defined at the bucket level](/articles/buckets.html).

All quorum values can be either passed as numeric values (longs), `com.basho.riak.client.cap.Quora` or `com.basho.riak.client.cap.Quorum`
instances.

### Other options

 * `:vclock`: TBD


## What to read next

 * [Secondary indexes](/articles/2i.html)
 * [Links and link walking](/articles/links.html)
 * [Map/Reduce](/articles/mapreduce.html)
 * [Integration with 3rd party libraries](/articles/integration.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

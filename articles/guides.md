---
title: "Welle, a Riak Clojure client: all documentation guides"
layout: article
---

## Guide list

[Welle documentation](https://github.com/clojurewerkz/welle.docs) is organized as a number of guides, covering all kinds of topics.

We recommend that you read these guides, if possible, in this order:


###  [Getting started](/articles/getting_started.html)

An overview of Welle with a quick tutorial that helps you to get started with it. It should take about
10 minutes to read and study the provided code examples

### [Connecting to Riak](/articles/connecting.html)

This guide covers:

 * Using Riak's HTTP transport with Welle
 * Using Protocol Buffers (PB) transport
 * Checking connection health


### [Working with buckets](/articles/buckets.html)

This guide covers:

 * What are Riak buckets
 * How to set/update bucket properties with Welle
 * How to list buckets
 * Bucket properties, what do they mean
 * How to fetch bucket properties
 * How to update bucket properties


### [Key/Value Operations](/articles/kv.html)

This guide covers:

 * How to store data in Riak with Welle
 * How to fetch data
 * How to delete data


### [Secondary indexes](/articles/2i.html)

This guide covers:

 * What are secondary indexes (2i)
 * Riak's approach to secondary indexes
 * Indexing stored values with Welle
 * Secondary index queries


### [Riak Search](/articles/search.html)

This guide covers:

 * Riak Search Overview
 * Indexing
 * Querying


### [Links and link walking](/articles/links.html)

This guide covers:

 * What are Riak links
 * Storing links with values
 * Link walking


### [Map/Reduce](/articles/mapreduce.html)

This guide covers:

 * What is map/reduce
 * Performing Map/reduce queries with Welle


### [Integration with 3rd party libraries](/articles/integration.html)

This guide covers:

 * Why integrate with other libraries?
 * Using welle.ring.store: [Ring session store](https://github.com/mmcgrana/ring/blob/master/ring-core/src/ring/middleware/session/store.clj) implementation on top of Riak
 * Using welle.cache: core.cache implementation on top of Riak


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

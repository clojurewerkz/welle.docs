---
title: "Welle, a Riak Clojure client: Using Map/Reduce"
layout: article
---

## About this guide

This guide covers:

 * What is map/reduce
 * Performing Map/reduce queries with Welle

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 1.0.x.


## Introduction

Riak’s primary querying and data-processing system is an implementation of the MapReduce programming paradigm [popularized by Google](http://labs.google.com/papers/mapreduce.html).
Key-value stores like Riak generally have very little functionality beyond just storing and fetching values.
MapReduce adds the capability to perform more powerful queries over the data stored in Riak.

Map/reduce queries consist of multiple *phases* that can be written in JavaScript or Erlang. They can either be stored in Riak buckets or
passed from the client. Riak also provides several built-in functions for common operations. Results from one
phase are piped into subsequent phases. Results from each phase may be accumulated into the final result or just used
as input for the next phase.

You can learn more about map/reduce phases in the [Riak documentation on map/reduce](http://wiki.basho.com/MapReduce.html).


## Performing Map/reduce queries

Map/reduce queries are performed using the `clojurewerkz.welle.mr/map-reduce` function:

{% gist 5677dba9c89fabed89ec %}

This query takes a bunch of numbers stored in a bucket as strings of text and computes the sum of them.

Map/reduce queries consist of one or more map and reduce *phases*. Lets take a look at one of the phases from the example above:

{% gist 90a5ed952ff227ef5ffc %}

This is a *map phase* that uses a built-in Erlang function called `map_object_value` in the `riak_kv_mapreduce` module (namespace). We instruct Riak to
not accumulate results from this phase, so mapped values won't appear in the final result.

Next we have two *reduce phases* using one built-in Erlang and one built-in JavaScript functions:

{% gist 4e813e8a6aa7a94eab39 %}

Final result returned by the `Riak.reduceSum` function is returned to the client.


## Wrapping up

Map/reduce queries in Riak are powerful but also fairly low level. They are suitable for processing large volumes of data thanks to the
good data locality: code is transferred to data, not the other way around.

Welle's support for map/reduce queries closely follows Riak's API. It may be possible for a higher-level API to emerge in the future,
when Welle developers and community gain more real world usage experience.


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read [Integration with 3rd party libraries](/articles/integration.html) next.



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

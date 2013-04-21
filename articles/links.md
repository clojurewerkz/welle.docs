---
title: "Welle, a Riak Clojure client: Using links and link walking"
layout: article
---

## About this guide

This guide covers:

 * What are Riak links
 * Storing links with values
 * Link walking

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 1.5.


## Introduction

[Riak Links](http://wiki.basho.com/Links.html) are metadata that establish one-way relationships between objects in Riak. They can be used to loosely model graph like
relationships between objects in Riak.

Much like [secondary indexes](/2i.html), links are specified à la carte, when a value is stored. It is then possible to traverse links (a process known as
**link walking**) to collect related values.


## Storing links with values

Links are specified using the `:links` option of `clojurewerkz.welle.kv/store` function. Each link is a map with three keys:

{% gist 4e80fec97bda82d376af %}

Bucket and key identify the value being referenced. Tag is a kind of relationship (some graph databases call it **relationship type**)

In the following example, we create a `friend` link to express friendship between two people:

{% gist 72f9f34cbf54017c7cef %}



## Link walking

To traverse a sequence links, use link walking DSL functions from the `clojurewerkz.welle.links` namespace:

{% gist 6878f674b9f7b7245429 %}

The `walk` function takes traversal starting point (specified here using the `clojurewerkz.welle.links/start-at` function) as the first argument and one or more steps as
remaining arguments. Each step is specified using the `clojurewerkz.welle.links/step` function and consists of

 * A bucket name
 * A link tag to use
 * Accumulator flag

If the accumulator flag is true, values found at the given step will be included in the end result. In our example above we want to find friends of
a particular person so we set the flag to `true`. If we wanted to find friends of friends of the person instead, we would use two steps and only included
values found during the last step into the final result set:

{% gist c31b47c1ab637713c102 %}


## Wrapping up

While Riak is not a graph database like [Neo4J](http://neo4j.org), links can cover many common scenarios, such as articles and comments or events and participants (when there are
reasons to not denormalize associated values into the parent).


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Map/Reduce](/articles/mapreduce.html)
 * [Integration with 3rd party libraries](/articles/integration.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

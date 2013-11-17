---
title: "Welle, a Riak Clojure client: Using Full Text Search with Riak and Welle"
layout: article
---

## About this guide

This guide covers:

 * Overview of Riak Search
 * Indexing data for searching with Welle
 * Querying Riak Search with Welle

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Welle does this guide cover?

This guide covers Welle 2.0, including development releases.


## Introduction

Riak started as a key-value store but over time accumulated several features that make querying it easier. One of the features is Riak Search.
Riak Search is a distributed, full-text search engine that is built on Riak Core and included as part of Riak open source. Search provides the most
advanced query capability next to MapReduce, but is far more concise; easier to use, and in most cases puts far less load on the cluster.


## Pre-requisites

Riak Search is an optional feature that can be disabled. Make sure it is enabled in the `app.config` file before attempting to use
it. Here is an example snippet that demonstrates how to do it:

``` erlang
 %% Riak Search Config
 {riak_search, [
                %% Riak Search is enabled
                {enabled, true}
               ]},
```


## Indexing

Before documents can be searched, they need to be **indexed**. Indexing is a process of taking a document with one or more fields, analyzing those
fields, producing data structures that can be efficiently searched over and storing them (in RAM, on disk, in a data store of some kind, etc).

Riak Search indexes values stored in Riak K/V using a precommit hook. Riak will use the content type of the stored value to determine
how to parse it. JSON, text, XML and Erlang term formats are supported. To enable Riak Search indexing for a bucket, pass
`:enable-search` option as true to `clojurewerkz.welle.buckets/update`. You can find more information about bucket
properties and how to update them in the [Working with Buckets guide](/articles/buckets.html).

It is possible to submit a document to Riak Search for indexing and not by adding it to a bucket via Riak K/V.
With Welle you do that via the `clojurewerkz.welle.solr/index` function which takes an index name and a document:

``` clojure
(require '[clojurewerkz.welle.solr    :as wsolr])

;; indexing
(wsolr/delete-via-query "an-index" "text:*")
(wsolr/index bucket-name {:username  "clojurewerkz"
                          :text      "Elastisch beta3 is out, several more @elasticsearch features supported github.com/clojurewerkz/elastisch, improved docs http://clojureelasticsearch.info #clojure"
                          :timestamp "20120802T101232+0100"
                          :id        1})
```

`clojurewerkz.welle.solr/delete-via-query` is a function that removes documents that match the query from the index.


### Search Schema

When a document is indexed, Riak Search will analyze and store it. **Analysis** is a process that has several stages:

 * Tokenization: breaking field values into **tokens**
 * Filtering or modifying tokens
 * Combining them with field names to produce **terms**

How exactly a document was analyzed defines what search queries will match (find) it. Riak Search is built on [Apache Lucene](http://lucene.apache.org)
offers several analyzers developers can use to achieve the kind of search quality they need. For example, different languages
require different analyzers: English, Mandarin Chinese, Arabic and Russian cannot be analyzed the same way.

How exactly a document was analyzed is defined by **search schema** assiciated with the index. With Riak, it is defined as an Erlang file and
then submitted using the `search-cmd` command line tool that comes with Riak. It is not necessary to define search schema to get
started or if all the fields in the documents are strings (text). For range queries over numerical or date values, it is
necessary to tell Riak Search how to analyze each document by defining the search schema.

Learn more in the [Riak Search documentation](http://wiki.basho.com/Riak-Search---Schema.html).


### Indexing via Command Line Tool

Riak comes with a command line tool, `search-cmd`, that can be used to index data, view search schema for a bucket, delete data from
the index, perform queries, bulk index data and more. Learn more in the [Riak Search documentation guide on indexing](http://wiki.basho.com/Riak-Search---Indexing.html).


## Querying

Riak Search supports term and field queries, boolean operators, lexographical range queries,
and trailing wildcard queries. Riak Search uses Lucene query syntax. For more information, see [Riak Search documentation on querying](http://wiki.basho.com/Riak-Search---Querying.html).

To query Riak Search, use the `clojurewerkz.welle.solr/search` function which takes an index name and a query and returns
a response as immutable Clojure map:

``` clojure
(require '[clojurewerkz.welle.solr    :as wsolr])

;; querying
(let [result (wsolr/search "an-index" "title:feature")
      hits   (wsolr/hits-from result)]
  (println result)
  (println hits))
```

To retrieve hits (documents found) from a response, pass it to the `clojurewerkz.welle.solr/hits-from` function.

`clojurewerkz.welle.solr/total-hits` is a function that takes a response and returns the total number of hits.
`clojurewerkz.welle.solr/any-hits?` is a similar function that returns true if the total number of hints is greater than zero,
false otherwise.


## Wrapping Up

Riak Search is a powerful feature that extends Riak's querying capabilities. It is not quite as extensive as
specialized full text search solutions such as Elastic Search and Apache Solr. It can, however, be very
effective in many common cases.

JSON, XML and text documents stored in Riak K/V can be indexed and queried.


## What To Read Next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Links and Link Walking](/articles/links.html)
 * [Map/Reduce](/articles/mapreduce.html)
 * [Integration with 3rd party libraries](/articles/integration.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Welle mailing list](https://groups.google.com/forum/#!forum/clojure-riak)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.

---
layout: post
title: "MongoDB Berlin 2013"
date: 2013-02-27 09:09
comments: true
categories: [conference]
---

Yesterday I attended the [MongoDB Berlin](http://www.10gen.com/events/mongodb-berlin-2013) conference which is a one day conference by 10gen to offer a space for collaboration and exchange around MongoDB. I want to sum up some (hopefully) interesting points for you. <!-- more -->

* be aware of pre-allocation done by MongoDB
* memory fragmentation is important to keep an eye on when dealing with updates on collections
* predict how your schema (although you have none in Mongo but your data still have some kind of structure) may change in future and how this will affect your sizing requirements
* collection.drop() can be significantly faster than removing a document from one collection (drop also reduces memory fragmentation because then you typically create a new collection from scratch)
* put different collections on different drives to scale write operations
* put the journal on an extra drive
* one index per query: you need to use compound indices if you need indices for several fields in a query
* try covered indices for performance improvements on queries you are doing quite often
* monitor with mongostat and mongotop
* when using mongostat interesting columns for inspect correct sizing, index setup and configuration are faults, locked, idx miss, qr, qw, ar, aw
* also nice web frontends for monitoring: [Server Density](http://www.serverdensity.com), the [MongoDB Monitoring Service](http://www.10gen.com/products/mongodb-monitoring-service) (free) and of course you can use tools like Nagios, Graphite or Ganglia
* have a look at the aggregation framework of MongoDB, looks quite promising
* [Spring Data](http://www.springsource.org/spring-data/mongodb) is the way to go for using MongoDB with Java due [Morphia](https://code.google.com/p/morphia) seems to be dead (or not much interaction in community) and [Hibernate OGM](http://www.hibernate.org/subprojects/ogm.html) is not worth to be mentioned at all

I can only recommend anyone of you who is interested in NoSql stores to try MongoDB. On [education.10gen.com](http://education.10gen.com) you can also sign up for several free online trainings.

Stay tuned.
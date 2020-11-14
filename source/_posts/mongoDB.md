---
title: mongoDB
date: 2020-11-14 16:57:27
tags: 数据库
---

MongoDB

<!-- more -->

Reading notes of 《MongoDB The Definitive GUide》 to learn how to ues the MongoDB


# Introduction

```
MongoDB is a powerful, flexible, and scalable general-purpose database.

-- chapter 1
```


## Ease of Use

MongoDB is a `document-orented database` , not a relational database.

the document-oriented approach makes it possible to represent complex hierarchical relastionships `with a single record`.

this fits naturally into the way developers in modern `object-oriented` think about thrie data.

there are also no predefined schemas: in MondoDB keys and values are `not fixed types or size schema`, this makes development `faster and quickly`


## Designed to Scale

as the data store grows. a defficult question: how should they scale their databases? 

scaling a database comes down the choice between scaling up (getting a bigger machine) or scaling out(partitioning data across more machines).

scaling up has drawbacks: large machines are very expensive.
scaling out has drawbacks: difficult administer

`MongoDB is designed to scale out`.  the document-oriented data model makes it easier to split data across multiple servers.



## Rich with Features

Mongodb is a general purpose database. so aside from creating reading updating and dalating data.

- indexing
    MongoDB supports generic secondary indexes and provides unique. compound, geospatial and full-text indexing capabilities as well.

- aggregate
    not understand...

- special collection and index types
    MongoDb supports time-to-live(TTL) collections for data that should expire at a certain time.

- File storage
    MongoDB supports an easy-to-use protocol for storing large files and file metadata( why sotring file to database...)

some features common to relational databases are not present in MongoDB, notably complex joins. MongoDB supports joins in a very limited way through use of the $lookuo in the 3.6 relase
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

`MongoDB is designed to scale out`. the document-oriented data model makes it easier to split data across multiple servers.

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

some features common to relational databases are not present in MongoDB, notably complex joins. MongoDB supports joins in a very limited way through use of the \$lookuo in the 3.6 relase

# getting started

some basic concepts of MongoDB

- A document is the basic unit of data for MongoDb
- A collection as a table with a dynamic schema
- A single instance of MongoDB can hot multiple independent database, each of which contains its own collections.
- Every document has a special key, "\_id" , that's unique within a collectin,(\_id is unique)
- MongoDB has a simple but powerful tool called the mongo shell

## Documents

an ordered set of keys with associated values. in javascript document are represented as object

```javascript
{"greeting": "hello world"}
```

the object can has multiple key/value , they can be one of several defferent data types

the keys in a document are strrings, any UTF-8 character is allowed in a key .

keys must not contain the character \0 this character used to signify the end of a key.

the . and % characters have some special preoperties.

MongoDB is type-sensitive and case-sensitive.

in MongoDB can't duplicate keys.

## Collections

A collection is a group of documents.

collections have dynamic schemas. this means that the documents within a single collection can have any number of different shapes.

so "Whey do we need separate collections at all ?", "Why should we use nore than one collections?"

reasons:

- keeping defferent kinds of documents in the some collection can be a nightmare for developers and admins.

- it's much faster to get a list of collections than to extract a list of the types of documents in a collections.

- grouping document of the some kind together in the same collection allows for data localiy.

- by putting only document of a single type into the same collection , we can index or collections or efficiently.

a collection is identified by its name. collections names canbe any UTF-8 string, with a few restrictions:

- not allow the empty string

- not allow end of the \0

- you should not create any collections with names that start with `system`, a prefix reserved for internal collections.

- user created collection should not contain the reserved chatacter % in their names.

a single instance of MongoDB can host serveral databases, each grouping together zero or more collections, A googd rule of thumb is to store all data for a single application in the same database , separate database are useful when stroing data for several applications or users on the same MongoDB server.

databsase names can be any UTF-8 string, with the following restrictions:

- not allow empty string

- can't contain any of thest chatcters : / \ . , " \* < > ; | ? % a single space or \0

- database name are case insensitive

- databse names are limited to maximum of 64 bytes.

there are some reserved database names(initialized in MongoDB installed):

- admin:
  the admin database plays a role in authentication and authorization.

- local
  this database stores data specific to a single server.

- congig
  sharded MongoDB clusters use the config database to store information about each shard

# Getting and Starting MongoDB

i am run MongoDB in docker .

when run with no aruments . MongoDB will use the default data directory(/data/db)

by default MongoDB listens for socket connections on port 27017.

## introdctioon to eht MongoDB Shell

MongoDB comes with a javascript shell that allows interaction with a MongoDB instance form the command line.

## Basic Operations with the Shell

CRUD command

### CREATE

the insertOne function adds a document to a collection.

```js
t = { name: "roger" };
db.user.insterOne(t);
```

### READ

find and findOne can be used to query a collection.

```js
db.user.find();
db.user.findOne();
```

### UPDATE

updateOne

```js
db.user.updateOne({ name: "roger" }, { $set: { age: 18 } });

WriteResult({ nMatched: 1, nUpserted: 0, nModified: 1 });
```

### DELETE

deleteOne and deleteMany permanently delete documents for the database.

```js
db.user.deleteOne({ name: "roger" });
```

can use deleteMany to delete all documents matching a filter

# Data Types

MongoDB supported types. MongoDB use the JSON , so JSON inclued six types: null boolean number string array and object.

this list following is other MongoDB supported types:

- Data
  MongoDB stores data as 64-bit integers representing milliseconds since the Unix epoch the time zone is not stored:

  ```js
  {
    x: new Data();
  }
  ```

- Regular expression
  Queries can use regular expressions using javascrip regular expression syntax

  ```js
  {
    x: /foobar/i;
  }
  ```

- Embedded document
  document can contain entire documents embedded as values in parent document

  ```js
  {
    x: {
      foo: "bar";
    }
  }
  ```

- Object ID
  an object ID is a 12 byte ID for documents;
  ```js
  {
    x: ObjectId();
  }
  ```
- Binary data
  Binary data is a string of arbitrary types. it connot be manipulated form the shell. Binary data is the only way to save non-UTF-8 strings to the database

- Code
  MongoDB also make it possible to store arbitrary javascript in queres and document
  ```js
  {x: function(){}}
  ```

## Datas

in javscript the data class is used for MongoDB data type, when creating a new Data Object, always call new Data(), return a string representation of the data , not an actual Data object.

## Array

array are value that can be used interchangeably for both orderd operations unordered operations

```js
{"things": [1, "a"]}
```

one of the great things about array in ducoment is that MongoDB "understands" their structure and knows how to reach inside of arrays to perform operations on their contents.

## Embedded Documents

a document can be used as the value for a key. this is called an embedded document. Embedded documents can be used to organize data in a more natural way than just a flat structure of key/value pairs.

## \_id and ObjectIds

every document stored in MongoDB must have an \_id key. the \_id key value can ben any type, but it default to an ObjectId, in a single collection ,every document must havue a unique value for \_id.

## OBJECTIDS

objectId is the defult type of \_id, the objectId class is designed to be lightweight while still being easy to generate in a globally unnique way across different machines.

## AUTOGENERATION OF \_ID

as stated earlier, if there is no \_id key present when a document is inserted one will be autmatically added to the inserted document. this can be handle by the MongoDB server but will generally be done by the driver on the client side

# Using the MongoDB shell

how to connect a MongoDB.

in mongo shell can input `help` to get the MongoDB help tips


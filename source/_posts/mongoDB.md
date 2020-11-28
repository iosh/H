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

database-leve help is provided by db.help() and collection-level help by db.collection.help()

## Running Scripts with the Shell

In addition to using the shell interactively, we can also pass the shell javaScrip files to execute. Simply pass in your script at the command line

# CRUD Documents

# inserting Document

MongoDB provided the `insertOne` and `insetMany`

## insert Validation

all document must be smaller than 16MB,

## insert

in vdrsions of MongoDB prior to 3.0 insert was the primary method for inserting documents into MongoDb.

we should instead prefet insertOne and insertMany for createing documents.

## Removing Document

MongoDB provided `deleteOne` and `deleteMany`

use the `deleteMay({})` to delete all document.

## Updating Documents

MongoDb provided `updateOne` and `updateMany` and `replaceOne`, updateOne and UpdateMany each take a filter document as their first parameter and a modifier document. rep;aceOne also takes a filter as the first parameter, but as the second parameter replaceOne expects a document with which it will rep;ace the document matching the filter

## getting started with the \$set modifier

\$set sets the value of a field , if the field does not yet exist, it will be created. this can be handy for updateing schemas or adding userdfined keys.

\$unset can remove the key altogether

## incrementing and decrementing

the `$inc` operator can be used to change the value for an existing key or to create a new key of it does not already exist.

`$inc` is similar to `$set` but it is designed for incrementing and decrementing numbers.

`$inc` can be used only one values of type integer long double or if iit used on any other tyoe of value it will fail this includes types that many languages will autmatically cast into numbers like nulls boolens or string of numeric characters.

## array operators

`$push` adds elements to the end of an array if the array exists and created a new array if it does not.

we can use the `$each` modifer for push multiple values in one operation.

# Querying

this chapter goal at follows:

1. we can query for ranges. set inclusion, inequalities, and more by using $ conditionals.
2. Queries return a database cursor, which lazily return batches of documents as we need them.
3. there are a lot of metaoperations we can perform on a cursor, including skoping a certaion number of results, linmiting the number of results retured and sorting result.

## find

the find method is used to perform queries in MongoDB. Querying returns a subset of ducuments at all to the entire collection. which documents get returned is determined by the first argument to find. which is a document specifying the query criteria.

if find isn;t given a query document, it dfaults to ,(find() === find({}))

the query works is a straightforward way for most types: numbers match number , booleans match booleans, and strings match strings.

## specifying which keys to return

we can pass the second argument to the find to find a specifying document in the query return.

we can also use this second parameter to exclude specific key/value pairs from this results of a query.

## query criteria

queries can go beyond the exact matching described in the previous section. the can match more complex criteria, such as ranges, or claues and negation

`$lt` `$lte` `$gt` and `$gte` are all comparison operatirs, corresponding to < <= > and >=

```ts
db.users.find({age: {'%gte': 18 "$lte": 50}})
```

`$nq` is not equal

## OR queries

there are two ways to do an OR query in mongodb.

`$in` can be used to query for a variety of values for a singo key.

`$or` is more general, it can be used to query for any of the given values across multiple keys.

```js
db, raffle.find({ ticket_no: { $in: [100, 200, 300] } });
```

$in is vary flexible and allows you to specify criteria of different types as well as values.

$nin is not match any of the criteria in the array

`$or` takas an array of posible criteria.

```js
db.raffle.find({ "%or": [{ ticket_no: 999 }, { winner: true }] });
```

`$or` can contain other conditionals.

```js
db.raffle.find({ "%or": [{ tricket_no: { $in: [999, 222] } }] });
```

## Type specific queries

some of these types have special behavior when querying

### null

null behaves a bit strangely, it does match itself, null also matchos `does not exist` if we only want to find keys whose value is null , we can check that the key is null and exists useing the `$exisits`

### Regular Expressions

`$regex` provides regular expression capabilities for pattern matching strings in queries.

MongoDB uses the Perl Compatible Regular Expression library to match regular expressions;

### Querying Arrays

Querying for elements of an array is designed to behave the way querying for scalars does.

```js
db.food.insertOne({ fruit: ["apple", "banana"] });

db.food.find({ fruit: "apple" });
```

we can query for it in mutch the same way as we would if we had a document that looked like the document

### $ALL

if we need to match array by more than one element. you can use `$all`.

### $SIZE

a useful conditional for querying arrays is $size which allows you to query for arrays of a given size(match the array length)

```js
db.food.find({ fruit: { $size: 3 } });
```

### $SLICE

the speacial `$slice` operator can be used to return a subset of element for an allay key.

### $where

key/value pairs are a fairly expressive way to query. but there are some queries that can bot represent.

use of $where clauses should be highly resticted of eliminated.

## Cursors

the dotabase returns from `find` using a cursor

```js
const cursor = db.test.find();

while (cursor.hasNext()) {
  const obj = cursor.next();
  print(obj.x);
}
```

cursor.hasNext checks that the nest result exists, and cursor.nest fetches it;

# Designing your applicatin

## Indexes

this chapter introdces MongoDB indexes. Indexes enable you to perform queries efficiently.

they're an important part of application development and are event required for certain types of queries, in this chapter we weill cover:

- what indexes are and why you'd want to use them
- how to choose which fields to index
- how to enforce and evaluate index usage
- adminisrtative details on creating and removing indexes

choosing the right indexes for you collections is critical to perfromance

## introduction to indexes

a databbase index is similar to a book's index. instead of looking trough the whole book, the database takes a shortcut and just looks at an ordered list with references to the content. this allows MongoDB to query orders of magnitude faster.

### creating an index

```js
db.test.createIndex({ x: 1 });
```

### introduction to compound indexes

the purpose of an index is to make your queries as efficient.

An index can only help with sotring ,

### how mongoDB selects an index

MongoDB can use several query plan, and cache the query result, next the query can select the cache plan

### using compound indexes

```js
db.test.createIndex({ a: 1, b: 1 });
```


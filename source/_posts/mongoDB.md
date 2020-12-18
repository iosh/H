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

# Indexes

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

when designing a compound index:

- keys for equality filters should appear first.

- key used for sorting should appear befor multivalue fields.

- key for multivalue filters should appear last

## explain output

explain gives you lots of information about the queries. it's one of the most important diagnostic tools there is for slow queries.

for any query , we can add a call to explain at the end.

there are tow types of explain output that you'll see most commonly: for indexed and nonindex queries.

## special index abd cikkectuibs types

this chapter covers the special collections and index tyes mongoDB hash available ,including:

- Capped collections for queue-like data

- TTL indexes for caches

- Full-text indexes for simple string searching

- Geospatial index for 2D and spherical geometries

- GridFS fro storing large files

### Geospatioal indexes

MongoDB has two types of geospatial indexes: 2dsphere and 2d.

2dsphere indexs work with spherical geometries that model the surface of the earth based ont the WGS84
datum.

### Indexes for Full Text Search

text indexes in mongoDB support full-text search requirements.

this type of text index should not be confused with the mongoDB Atlas Full-Text Search Indexes. wtich utilize Apache Lucene for additional text search capbilities when compared to mongoDB text indexes. use a text index if you application need to enable users to submit keyword queries that should match titles ,decriptions and text in other fields within a collection.

string will be tokenized and stemmed and the index updated in. potentially, many place, for this reason , writes involving text indexes are usually more expensive than writes to single-fild, compound, or enven moltikey index.

thus, weill tend to see poorer writer performance on text-indexed collections than on tohers, they will also slow down data movement if sharding : all text must be reindexed when it is migated to a new shard.

#### create a text index

```js
db.test.createIndex({ title: "text", body: "text" });
```

this is not like a normal compound index where there is an ordering on the keys. by default, each is given equal consideration in a text index.

we can control the relative importance mongoDb attaches to each field by specifying weights:

```js
db.text.createIndex(
  { title: "text", body: "text" },
  { weights: { title: 3, body: 2 } }
);
```

we cannot change field weights after index creation .

for some collections, you may not know which fields a documents will contain, so we can create a full-text index on all string fields in a document by create an index on $\*\* this not only indexes all top-;level string fields , but also searches embedded documents and array for dtring fields

#### text search

use the $text query operator to perform text searches on a collection with a text index.

```ts
db.test.find({ $text: { $search: "hello word" } });
```

#### optimizing full-text search

there are a couple of way sto optimize full-text searches .

if you can first narrow you search result by other criteria, you can create a compound index with a prefix of those criteria and then the full-text fields

```ts
db.test.createIndex({ data: 1, post: "text" });
```

#### searching in other languages

we can set the text indexes a default_language opthin

```ts
db.test.createIndex({ title: "text" }, { default_language: "chinese" });
```

```ts
db.test.inster({title: '你好'}， language: 'chinese')
```

## capped collections

mongoDB normal collections are dynamcally and automatically grow in size to fit additional data.

mongoDB also supports a different type of collections, called a capped collection wich is created in advace and is fiexd in size

having fixed-size collections brings up an interesting questions: what happens when we try to insert into a capped collection that is already full?

the answer is that capped collections behave like circular queuess: if we're out of space the oldest document will be deleted, and the new one will take its lace.

### creating capped collections

```ts
db.createCollection("mycappedCollection", { capped: true, size: 100000 });
```

the command create a capped collection. mycappedCollection that has fixed size of 100000 bytes, that also specify a limit on the number of document is a capped collectios:

```ts
db.createCollection("test", { capped: true, size: 1000, max: 10 });
```

# Introduction to the Aggregation Framework

many applications require data analysis of one form or another . mongoDB provides powerful support for running analytice natively using the aggregation framework. in this chapter we introduce this framework and some of the fundamental tools it provides we'll cover;

- the aggregation framework

- aggregation stages

- aggregation expressions

- aggregation accumulatiors

```ts
db.companies.aggregate([
  { $match: { founded_year: 2004 } },
  {
    $project: {
      _id: 0,
      name: 1,
      founded_year: 1,
    },
  },
]);
```

the command to match the founded_year of the document and use the pipeline to reduce the output to just a few fields per document. will exclude the \_id and include the name and founded_year

many pipeline

```ts
db.test.aggregate([
  { $match: { founded_year: 2000 } },
  { $sort: { name: 1 } },
  { $limit: 5 },
  { $project: { _id: 0, name: 1 } },
]);
```

## expressions

the aggregation framework supports many different classes of expressions:

- boolean expressions allow us to use AND OR and Not expressions

- set expressions allow us to work with arrays as sets, in particular, we can get the intersection or union of two or more sets, we can also take the difference of two sets and perform a number of other set perations.

- comparison expressions enable us to express many different types of range filters

- arithmetic expressions enable us to calculate the ceiling, floor natureal log and log, as well as perform simple arthmetic operations like multiplication, division, addition , and subtraction, we can event do more complex operations, such as calculationg the square root of a value.

- string expressions allow us to concatenate find substrings,and preform operations having to do with case and text search operations.

- array expressions provide a lot of power for manipulating arrays, including the ability to filter array elements. slice an array, or just take a range of values form a specific array.

- variable expressions which we won't dive into too deeply, allow us to work with literals, expressions for parsing date values, and conditional expressions.

- Accumulators provide the ability to calculate sums, descriptive statistics, and many other types of values.

## $unwind

when workin with array fields n an aggregation pipeline.

1. match - match some document
2. unwind - handle previous documents
3. project - handle previous documents

```ts
db.test.aggregate([{$match: someName: "somevalue"}, {$unwind: $someKey}, {$project: {_id: 0, name: 1, somekey: 1
}}])

```

## Array Expressions

$filter operator is designed to work with array fields and specifies the options we must supply .

selects a subset of an array to return based on the specified conition , returns an arry with only those elements that match the condition. the returned element are in orginal order.

## Accumulators

the Accumulatro provides enable us to proform operations such as summing all values in al particular field ($sum). calcalation an average ($avg), etc, we also consider $first and $last to be accumulatiors bacause these consider values in all documents that pass through the stage in which they are used. $max and $min are tow more examples of accumulatros that consider a stream of documents and save just one of the values they see. we can use $mergeObject to combine multiple document into a single document.

we also have accumulators for arrays, we can $push values onto an array as documents pass through a pipeline stage. $addToSet is very similar to $push except that is ensures no duplicate values are included in the resulting array.

## introduction to grouping

the group stage performs a function that is similar to the SQL GROUP BY command. in a group stage ,we can aggregate together value form multiple documents and perform some type of aggregation ioertation on them. such as calculating an average

# Transactions

Transactions are logical groups of processing in a database, and each group or transaction can contain one or more operations such as reads and/or writers across multiple documents. mongoDB supprts ACID-compliant transactions acroos multiple operations , collections, databases, documents, and shards, in this chapter , we introduce transactions , define what ACID means for a database, highlight how you use these in you applications, and provide tip for tuning transactions in mongoDB . cover following:

- what a transaction is

- hot to use transactions

- tuning transaction limits for you application

## introduction to transactions

as we mentioned above. a transaction is a logical unit of processing in database that includes one or more database operations, which can be read or write operations, there are stuations whre your application may require reads and writes to multiple documents as part of this logical unit of precessing, an improtant aspect of a transaction is that it is never partially completed it either succeeds or fails.

## a definition oa ACID

ACID is the accepted set of properties a transaction must meet to be a true transaction, ACID is an acronym for Atomicity Consistency isolation and Durability. ACID transactions guarantee the validity of you data and of you database's state even power failures or other errors occur.

Atomicity ensures that all operations inside a transaction will either be applied or noting will be applied. a transaction can never be portially applied, either it is committed or aborts.

Consistency ensures that if a transaction succeeds, that database will move form one consisitent state to the next consisint state.

Isolation is the property that permits multiple transactions to run at the same time in your database . it guarantees that a transaction wile not view the partial results of any other transaction, which means multiple transactions will have the same results as running each of the ransactions sequentially.

Durability ensures that when a transaction is coommitted all data will presist event in the case of a system failure

## How to Use Transactions

MongoDb provides two APIs to use transactions. The first is a similar syntax to relational databases(example: start_transaction and commit_transaction) and called the core API and the second is called the callback API, which is the recommended to using to using transactions.

the core API does not provide retry logic for the majority of errors and requires the developer to code the logic for the operations. the transaction commit function and any retry and error logic required.

the callback API provides a single function that wraps a large degree of functonality when compared the core API including staring a transacion assocated with a specified logical session , execution a function supplied as the callback function , and then committing the transaction (or aborting on error) this funciton also includes retry logic that handle commit errors, this callback API was added in MongoDB 4.2 to simplify application develppment with transactions as well as make it easier to add application retry logic to handle andy transaction error.

in both APIs, the developer is responsiblefor staring ht logical session that will be used by the transaction. Both APIs require operations in a transaction to be associated with a specific logical seeion . aA logical session in MongoDB tracks the time and sequencing of the operations in the context of the entre MongoDB deployment, A logical seeion or sever seeion is part of the underlying framework used by client sessions to suuport retryable writers and causal consistency in MongoDB both of these features were added in MongoDB version 3.6 as part of te foundation required to support transactions,, a specitfuc squence of read and write operations tha have a cauale relationship relected by theri ordering is defined as a causally consistent client session in mongoDB . a calient session is started by an applicaition and used to interact with a server session

| Core API                                                                                                                                                                                           | Callback API                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Requires explicit call to start the transaction and commit the transaction                                                                                                                         | Starts a transaction, executes the spectified operation, and commits (or aborts on erro)                          |
| Does not incorporate error handling logic for transientTranactionEroor and UnkonwoTranasctinCommitResult, and instead provides the filxbility to incorporate custom error handling for these error | Automatically incorporeate error-handling logic for TransientTransactionError and unkonwnTransaction-commitResult |
| Requires explict logical sesion to be passed toAPI for the specific trasaction                                                                                                                     | Requires explicit logical session tobe pased to API for the specific transaction                                  |

# Application Design

- Schema design considerations

- Trade-offs when deciding whether to embed data or to reference it

- Tips for optimization

- Consistency consideration

- How to migrate schemas

- How to manage schemas

- When MongoDB isn't a good choice of data store

## Schema Design Considerations

A Key aspect of data representation is the design of the schema. which is the way your data is represented in your documents. the best approach to this design is to represent the data the way your application wants to see it . Thus , unlike in relational databases, you first need to understand your queries and data access patterns brfore modeling you scahema

here are the key aspects you need to consider when designing a schema:

### Constraints

you need to understand any database or hardware limitations.. you also need to consider a number of MongoDB's specific aspects. such as the maximum document size of 16 MB, that full documents get read and written from disk, that an update rewrites the whole docment , and that atomic updates are at the document level

### Access patterns of your queries and of your writes

you will need to identify and quantify the workload of your application and of the wider system. the workload encompasses both the reads and the writes in your application. once you know when queries are running and how frequently, you can identify the most common queries. these are the queries you need to design your schema to supprot. once you have identified these queries you should try to minimize the number of queries and ensure in your design that data that gets queried together is stored in the same document.

Data not used in these queries should be put into a different collections.Data tha is infrequently used should also be moved to a different collection, it is worth considering if you can separate your dynamic (read/write) data and your staic data. the best preformance results occur when you priorize your schema design for your most common queries.

### Relation types

you should consider which data is related in terms of your application's needs, as well as the relationships's. needs, as well as the relationships between documents, you can then deteremine the best approaches to embed or reference the data or documents. you will need to work out how you can reference documents without having to perform addtional queries, and how many documents are updated when there is a relationship change, you must also consider if the data sthructure is easy to query , such as tith nested array, thich suprot modeling certain relationships.

### Cardinality

once you have determined how your documents and your data are related , you should consider the cardinality of these relationships specifically is it one to one , one to many many to many, ont to minlions or many to bilions

it is very improtant to establish the cardinlity of the relationships to ensure you use the best format to model them in your MongoDB schema You should also consider whether the object on the many/monlions side is accessed sparately or only on the context of the parent object as well as the ratio of pdates to reads for the data field in question, the answers to these questions will help you to detemine whether you should embed document or reference documents and if you should be denmoealizing data acoss documents.

## Schema Design patterns

Schema design is important in MongoDB.

Scheme design patterns that migth apply include:

- polymorphic pattern

this is suitable where all documents in al collection have a similar, but not identical, strcutre, it involves identifying the common fields across the documents that support the common queries that will be run by the appliction, tracking specific fields in the documents or subdocuments will help identify the diffrerences between the data and different code paths or claaes/ subclasses that can be coded in you application to manage these differences. this allows for the use of simple queries in al single collection of not quite -identical documents to improve quer performance.

- Attribute pattern

this is suitable when there are a subset of fields in a document that share commonfeatures on which you want to sort or query, or when the fields you need to sort on only exist in a subset of the documents, or when both of these conditions are true, it involves reshaping the data into an array of key/value pairs and creating an index on the elements in this array, Qualifirers can be added as additional fields to these key/value pairs this pattern assists in targeting many similar fields per document so that fewer indexes are required and queries become simpler to write.

- Bucket pattern

this is suitable for time series data where the data is captured as a stream over a period of time. it is much more efficient in MongoDB to 'bucket' this data into a set of documents each holding the data for a particular time range than it is to create a doument per point in time/data poing. for example you might use a one-hour buket and place all readings for that hour in a array in a single document . the document tiself will have start and end times indication the period this bucket covers.

- Outlier pattern

this addresses the rare instances where a few queries of documents fall outside the noremal partten for the application . it is an advanced schema pattern designed for situations wheere popularity is a factor . this can be seen in social netwoks with major influences, book sales, movie reviews etc it uses a flage to indicate the document is an outlier and stores the addtitonal overflow into one or monre documents that refer back to the firest document via the \_id the flage will be uesd by your applicaiton code to make the additonal queries to retrieve the overflow documents.

- Computed pattern

this is used when data needs to be computed frequently, and it can also be used when the data access pattern is read-intensive. this pattern recommends that the calculations be done in the backround , whith the main document being update peridically. this provides a valid approximation of the computed fields or document without having to continuousl generate these for individual queries, this can significantly reduce the starin on the CPU by avoiding repetition of the same calulations, particularly in use cases where reads trigger the calculation and you have a high read-to-write ratio.

- Subset pattern

this is used when you have a working set that exceeds the available RAM of the macehine. this can be caused by large documents that contain a lot of information that isn't being used by your application. this pattern suggests that you split frequently used data and infrequently used data into two separate collections. A typical example mingth be an ecommerce applicaiton keeping to 10most recent reviews of a prooduce in the main requentl accessed collection and moveing all the older reviews into a second collection querid oly of the application need more than the loast 10reviews.

- Extended Reference pattern

this is used for scenarios where you have many different logical entities or things. each with their own collection, but you may want to gether these entities together for a specific function . A typical ecommerce shema might have separate collectins for orders customers and inventory. this can have a negative performance impact when we want to collect together all the information for a single order from these separate collections. the solution is to identify the frequenly accessed fields and duplcate these within the order this would be the name and address or the customer we are shipping the item to . this pattern trades off the number of queries necessary to collate the information together.

- Approximation pattern

this is useful for situation where resouce expensive (time memory CPU cycles) calculations are needed but where exact is not absolutely required, an example of this is an image or post like/love counter or a pag view conunter where knowing the exact count isn't necessary. in these situations aapplying this pattern can applying this pattern can greatly reduce the number of writers for exmlo by only updating the counter agter every 100 or more views instead of after every view.

- Tree pattern

this can be applied when you hanve a lot of queryies and have data theat is parmarily hierarchical in structure , it follows the earlier concaept of storing data together that is typilcally queryied together , in MongoDB you can easily sotre a hierarchy in an array within the same documents, in the exple of the ecmmoerce site , specifically its produce catelog, there are ofen producets that belong to multiple cate gories or to categoris thea are part of other cateres .

- preallocation pattern

this was primarily used with the MMAP storage engine, but there are still uses for this pattern . the pattern recommmends createing an initial empty structure that will be populated later.

- Document versioning pattern

this provides a machanism to enable retention of older versions of documents, it requires an extra field to be added to each document to track the documents version in the main collection, and an dadditional collection that contains all the revisions of the documents thie pattern make a few assumptions specficall, that each document has a limited number of vevisions. that there are not large numbers of documents that need to be versioned and that the queries are promarily done on the current version of each documents . in situations where these assumptions are not valid, you may need to modify the pattern or consider a different schema desion pattern

# Normalization Versus Denormalization

there are many ways to represent data. and noe of the most important issues to consider is how much you should normalize your data. Normalization refers to dividing up date into multiple collections with references between collections. Each piece of data lives in one collections. although multiple documents may reference it. thus t change the data only one documents must be upodated the MongoDB aggreation Framework offers joins with tthe $lookuo stage. which performs a left outer join by adding documents to the joined collection where there is a matching document in the source collection it add a new array field to each matched document in the joined collection with the datils of he doucment from the source ollectin these reshaped documents are then available in the next stage for further processing.

Denormalization is the opposite of normalization : embedding all of the data in a single document. instead of document containing references to one definitive copy of the data this means that multiple documents need to be updated if the information changes but enables all related data be fetched with a single query

Deciding when to normalize and when to denormalize can be difficult: typically normalizing makes writers faster and denormalizing makes reads fater thus you need to decide what trade offs make sensu for you application

conparison of embedding versus references

| Embedding is better for...                                     | References are better for...                   |
| -------------------------------------------------------------- | ---------------------------------------------- |
| Small subdocuments                                             | Large subdocuments                             |
| Date that does not change                                      | Volatile data                                  |
| When eventual consistency is accptable                         | When immediate consisitncy is necessary        |
| Documents that grow by a smail amount                          | Documents that grow by a large amount          |
| Data that you'll often need to perform a second query to fetch | Data that you'll often exclude from the result |
| Fast reads                                                     | Fast writers                                   |

## Cardinality

Cardinality is an indication of how many references a collection has no another collection. common relationships are one-to-one, one-to-many. or many-to-many.

# Optimization for Data Manipulation

to optimize your application, you must first datermine what its bottleneck is by evaluation its read and write performance. optimozomg reads gemerll involves having the correct indexes and retuning as much of the information as possible in a single document. optimizing writers usully invlues minimingzin the number of indexs you have and making updates as efficient as possible

## Removing Old Data

Some data is only important for a brief time: after a few weeks or months it is just wasting storage space . There are three popular optinins for removing old data: using capped collection, using TTL collections. and dropping collections per time period.

# Setting Up a Replica Set

MongoDB high-avaliability system: replica sets. it covers:

- what replica sets are

- How to set up a replica set

- What configuration options are available for relica set members

## Introduction to Replication

Since the first chapter , we've been using a standalone server. a single mongod server. it's an easy way to get started but a dangerous way to run in production . what if your server crashes or becones unavailable , your database will be unavailable for at least a little while . if there are problems with the hardware, you minght have to move your data to another machine. in the worst case disk or network issues could leave you with corrupt or inaccessible data

## Setting Up a Replica Set

we start run three mongodb look like this:

```sh
mongod --replSet mdbDefGuide --dbpath ~/data/rs1 --port 27017 \
    --smallfiles --oplogSize 200
mongod --replSet mdbDefGuide --dbpath ~/data/rs2 --port 27018 \
    --smallfiles --oplogSize 200
mongod --replSet mdbDefGuide --dbpath ~/data/rs3 --port 27019 \
    --smallfiles --oplogSize 200
```

then connects to ont of the running mongod instances

```sh
mongo --port 27017
```

then in the mongo shell . create a configuration document and pass the this to the rs.initiate() helper to initiate a replice set. this will initiate a replice set containing three members and propagate the configuration to the rest of the mongods so that a reploce set is formed:

```ts
rs.initiate({
  _id: "mdbDefGuide",
  members: [
    { _id: 0, host: "172.18.0.3:27017" },
    { _id: 1, host: "172.18.0.4:27018" },
  ],
});
```

there are several important parts of a relica set configuration document . the config's "\_id" is the name of the replica set that you passed in on the command line , make sure that this name matches exactly.

rs is a global variable that contains replication helper function (run rs.help()) to the the helpers it pexposes.

# Obeserving Replication

if your relica set elected the mongod on port 27017 as primary, hten the mongo shell used to initiate the relica set is currentlly connected to the primary

we can use the db.isMaster() command to show the status of the replica set. is a much more concise form tha rs.status() it is also a convenient means of determining which member is promary when writing application

if you attempt to query a secondary, you'll get an error staring that it's not the promary, this is to protect you application from accidentally connection to a secondary and reading stale data, to allow queries on the secondary , we can set an "i'am okay with reading from secondaries" flag like so

```ts
secondaryCoon = new Mongo("localhost:27018");
secondaryDB = secondaryConn.getDB("test");
secondaryCoon.setSecondaryOk();
```

the secondary does not accept the writer, it will only perform writes that it gets through relication, not from clicents.

automatic fail-over , if the primary goes down, one of the secondaries will automatically be elected parimary

there are a few key concepts to remember:

- Clients can send a primary all the same operations they could send a standlalone server(reads, writers ,commands, index builds)

- Clients cannot write to secondaries

- Clients, by default connot read from secondaries , you can enable this explicitly setting an "i know i'm reading from a secondary" setting on the connection.

# Changing your Replica Set Configuration

Replica set configurations can be changed at any time: members can be added removed or modified there are shell helpers for some common operations:

- rs.add

```ts
rs.add("localhost:27020");
```

- rs.remove

```ts
rs.remove("localhost:27020");
```

- rs.config() check a reconfiguration

```ts
rs.config();
```

each time you change the configuration , the “version” field will increase.

you can also modify existing members, not just add and remove them. to make modifications, create the configuration document that you want in the shell and call rs.reconfig

```ts
const config = rs.config();
config.members[0].host = "localhost:27022";
rs.reconfig(config);
```

# How to Design a Set

There are a couple of common configurations that are recommended:

- A majority of the set in on data center, This is a good design if you have a primary data center where you always want you replica set's primary to be located, so long as you primary data center in healthy , you will have a primary. however if that data center becomes unaveliable , your secondary data center will not be able to elect a new primary.

- An equal number of servers in each data center , plus a tie-breaking server in a third location , this is a good design if your data centers are equal in preference since fenerall servers from either data center will be able to see a majority of the set however it invlove having three separate locations for servers.

# Component of a Replica Set

this chapter covers how this pieces of a replica set fit together including:

- How replica set members replicate new data

- How bringing up new members works

- How elections work

- Possible server and network failure scenarios

## Syncing

Replication is concerned with keeping an identical copy of data on multiple servers. the way MongoDb accomplishes this is by keeping a log of operations , or oplog. contaning ever writer that a primary performs this is a capped collection that lives in the local database on the primary the secondaries query this collection for opeartions to replicate

Each secondary maintains its own oplog recording each operation is relicates from the primary. this allows any member to be used as a sync source for any other member , secondaries fetch operations from the member they are syncing from . apply the operations to their dataset and then write be operation to their oploog if applying an operation fails the secondary will exit

## Heartbeats

Members need to know about the other members states who's primary, who they can sync from , and who's down, to keep an up-to-data view of the set, a member sends out a heartbeat request to every other member of the set every two seconds, a heartbeat request is a short message tha checks everone's state

## Nenver States

members also communicate what state they are in via hearbeats, we've already discussed two states primary and secondary thre are several other normal states that you'll often see members be in:

- STARTUP
  the member isfirst started. while MongoDB is attempting to load its replice set configuration.

- STARTUP2
  this state lasts throughout the initial sync process.

- RECOVERING
  this tate indicates that the member is operation correctly but is not available for reads.

- ARBITER
  ARbiters have a special state and should alwayys be in the state druing normal opration

- DOWN
  if a member was up but then becomes unereachablel. it will enter this state. note that a member repported as down might in fact still be up just unreachable doue to network issues.

- UNKNOWN
  if a member has never been able to reach another member, it will not know what state it's in, so it will report it as UNKOWSN this generally indicates that the unkonwn member is down or that there are network problems between the two members.

- REMOVED
  this is the state of a member that hash been removed from the set if a removed member is added back into the set it will transition back into its normal state.

- ROLLBACK
  this tate is used when a member is rolling back data .

## Elections

a member will seek election of it cannot reach a primary , a member seeking election will send out a notice to all of the members it can reach these members may know why thi memebr is an unsuitable ppromiary it may be behind in replication or there many alread be a primary that the member seeking election cannot reache in these cases, the other memebers will vote against the candidate.

# Connection to a Replica Set from Your Application

this chapter covers how applicaitons interact with replica sets, including:

- How connections and failovers work

- Waiting for replicaiton on writes

- Routing reads to the correct member

## Client-to Replica Set Connnection Behavior

MongoDB client libaries are designed to manage communication with MongoDB servers regardless of whether the server is standalone MongoDB instance or a replica set, for replica sets, by default drivers will connect to the primary and rout all traffic to it , you application can perform reads and writes as though it were talking to sandalon server while your replica set quietly keeps hot standbys ready in the background

```ts
`mongodb://server-1:27017,server-2:27017,server-3:27017`;
```

# Wating for Replication on Writes

Depending on the needs of you application , you might want to require that all writes are replicated to a majority of the replica set before they are acknowledged by the server. in the rare circumstance where the promary of a set goes down and the newly elected promary primary did not relicate the very last writes to the former primary, those writers will be rolled back when the former primary comes back up, they can be recovered but it requires manual inter vaention for many appliaciton having a small number or writes rolled back is not a problem , in a bolg appliaciton for example there is litter read dinger in rolling back one or two comments from ont reader

# Sharding

## Interdoction to Sharding

- What sharding is and the components of a cluster

- How to configure sharding

- The basics of how sharding interacts with your application

### Waht is sharding

sharding refers to the process of splitting date up across machines; the term partitioning is also sometimes used to describe this concept. By putting a subset oa date on each machine , it becomes possible to shore more data and handle more load without requireing larger or more powerfull machines just a larger quanitiy of less powerful machines, sharding may be used for other puposes as well ,including placing more frequently accesssed data on more performant hardware or splitting a dataset based on geograplhy to locate a subset of documents in a collction close to the application servers from which they are most commonly accessed.

MongoDB supports autosharding, which tries to both abstract the archiecture away from the application and simplify the administration of such a system.

## Understanding the components of a cluster

MongoDB's sharding allows you to create a cluster of many machines and break up a collection across them, putting a subset of data on eache shard this allows your application to grow beyond the resource limits of a sandalone server or replica set

## Sharding on a Single-Machine Cluster

start cluster on a single machine.

```sh
mongo --nodb --norc
```

to create a cluster, use the ShardingTest class.

```ts
st = ShardingTest({
  name: "one-min-shards",
  chunkSize: 1,
  shards: 2,
  rs: {
    nodes: 3,
    oplogSize: 10,
  },
  other: {
    enableBalancer: true,
  },
});
```

now use the new mongo shell

```ts
// use the mongo shell , not connect the DB
mongo --nodb
```

```ts
db = new Mongo("localhost:20009").getDB("accounts");

for (let i = 0; i < 10000; i++) {
  db.user.insert({ name: "user" + 1, create_at: new Date() });
}

// now the collection hash 10000 user

db.user.count(); // 10000
```

```ts
// show the sharding status
sh.status();
```

# Configuring Sharding

- How to set up config servers, shards, and mongos processes

- How to add capactty to a cluster

- How data is stored and distrubuted

## When to shard

deciding when to shard is a balancing act. you generally do not want to shard too early because it adds operational complexity to you deployment an forces you to make design decisions that are difficult to change later. on the other hand. you do not want to wait too long to shard because it is difficult to shard an overloaded system without downtime.

in general, sharding is used to:

- Increase available RAM

- Increase available disk space

- Reduce load on a server

- Read ot write data with greater throughput than a single mongod an hanlde


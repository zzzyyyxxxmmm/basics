# Advantage Of MongoDB

### Ease of Use
MongoDB is a document-oriented database. The primary reason for moving away from the relational model is to make scaling out easier, but there are some other advantages as well.

A document-oriented database replaces the concept of a “row” with a more flexible model, the “document.” By allowing embedded documents and arrays, the document- oriented approach makes it possible to represent complex hierarchical relationships with a single record. This fits naturally into the way developers in modern object- oriented languages think about their data.

There are also no predefined schemas: a document’s keys and values are not of fixed types or sizes. Without a fixed schema, adding or removing fields as needed becomes easier.

### Easy Scaling
MongoDB was designed to scale out. Its document-oriented data model makes it easier for it to split up data across multiple servers. MongoDB automatically takes care of balancing data and load across a cluster, redistributing documents automatically and routing user requests to the correct machines. This allows developers to focus on pro‐ gramming the application, not scaling it. When a cluster need more capacity, new ma‐ chines can be added and MongoDB will figure out how the existing data should be spread to them.

### others

MongoDB is intended to be a general-purpose database, so aside from creating, reading, updating, and deleting data, it provides an ever-growing list of unique features:

**Indexing**

MongoDB supports generic secondary indexes, allowing a variety of fast queries, and provides unique, compound, geospatial, and full-text indexing capabilities as well.

**Aggregation**

MongoDB supports an “aggregation pipeline” that allows you to build complex aggregations from simple pieces and allow the database to optimize it.

**Special collection types**

MongoDB supports time-to-live collections for data that should expire at a certain time, such as sessions. It also supports fixed-size collections, which are useful for holding recent data, such as logs.

**File storage**

MongoDB supports an easy-to-use protocol for storing large files and file metadata.

# Simple exmaple
```
mongo -u root -p example
```

```
> db
test
> post = {"title" : "My Blog Post",
... "content" : "Here's my blog post.",
... "date" : new Date()}
{
        "title" : "My Blog Post",
        "content" : "Here's my blog post.",
        "date" : ISODate("2020-02-28T01:58:58.002Z")
}

> db.blog.insert(post)
WriteResult({ "nInserted" : 1 })
> db.blog.find()
{ "_id" : ObjectId("5e58788fe4f33bbb8fc842f2"), "title" : "My Blog Post", "content" : "Here's my blog post.", "date" : ISODate("2020-02-28T01:58:58.002Z") }
> db.blog.update({title : "My Blog Post"}, post)
> db.blog.remove({title : "My Blog Post"})
```

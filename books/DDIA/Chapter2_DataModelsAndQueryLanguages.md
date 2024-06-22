# Data Models and Query Languages

 Data model show how software is written and how we think about the problem.
 The data format application developers give the database, and mechanism to ask again.

1. Applications are built by layering one data model on top of another, each layer needs to be represented by the next-lower layer.
2. Application developers model the world in term of objects, data structures and APIs.
3. Storage layer stores these data structures with data model like JSON, XML documents or tables in relational databases, or a graph model.
3. Database decides a way of representing the JSON/XML/relational/Graph model in bytes. This representation allows the data to be queried, searched, processed. 

This chapter focus on 3, a range of general-purpose data models for data stor‐ age and querying.

## Relationsal Model vs Document Model
- SQL/Relational Data Model - each relation is an unordered collections of tuples.
- NoSQL - driving forth are 
    - need for greater scalability, including very large dataset or high write throughput. 
    - specialized query operations that are not well supported by the relational model
    - desire for more dynamic and expressive data model.

Many to one or many to many relationship are more natural modeled in relational models, than document models. Most document model DBs don't have good join supports.

| Data Model | Pros | Cons |
| ------------|-----|------|
| Relational Model | Better handle joins, many-to-one or many-to-many relationship, The query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use. The choices are made automatically by the query optimizer, not by the application developer.| Unnecessarily complicated application code, poor storage locality (data split across multiple tables on disks) |
| Document Model | Schema flexibiity through **schema-on-read**/schemaless, data locality for queries | poor supports for join
---


#### Impacts on application code
- Largely depends on usage
- The relational technique of shredding— splitting a document-like structure into multiple tables can lead to cumbersome schemas and unnecessa‐ rily complicated application code.
- The document model has limitations: for example, you cannot refer directly to a nested item within a document, but instead you need to say something like “the second item in the list of positions for user 251”

#### Schema flexibility
- Schema-on-read - Document databases sometimes are called schemaless, which is misleading, as the code reads the data assume some kind of data structure. **A more accurate term is schema-on-read, aka, the structure of the data is implicit, and only interpreted when data is read, in contrast with schema-on-write when schema is defined explicitly and databases enforce is on write.** 
- The schema-on-read approach is advantageous if the items in the collection don’t all have the same structure for some reason (i.e., the data is heterogeneous)
- Schema evolution support in Chapter 4.

- Document and relational databases are now converging
    - Relational databases support JSON data type
    - Document databases support joins either via query languages, or drivers.

#### Data locality for queries
- A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB’s BSON). If your application often needs to access the entire document (for example, to render it on a web page), there is a performance advantage to this storage locality. 
- Writes are expensive unless it is in place for document model.
- Data locality also used in relational data model (Spanner), and column family (Big data model).

#### Convergence of document and relational databases
- Relational DB also support XML and JSON
- Document DB enhanced join support through DB (MongoDB) driver offering client side joins.

## Query Languages for Data 
- Declarative: describe what, leave the how to databases, more concise, hides implementation details, e.g. SQL. Easier to leverage CPUs' parallel execution. Preferred both in databases or web browsers.
- Imperative: requires programmer to outline the exact procedures and control flows for data retrieval, manipulations etc.

## Map-Reduce Query
- MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google [33]. A limited form of MapReduce is supported by some NoSQL datastores, including MongoDB and CouchDB, as a mechanism for performing read-only queries across many documents.
- In between declarative and imperative, the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework.

## Graph Data Models
When the connections within your data become more complex, it becomes more natural to start modeling your data as a graph.
- Property data model (vertex and edges) & Cypher query language
- Triple-store model (subject, predicate, object) and SPARQL query language




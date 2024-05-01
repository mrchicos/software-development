# Data Models and Query Languages

 The data format application developers give the database, and mechanism to ask again.

- Applications are built by layering one data model on top of another, each layer needs to represented by the next-lower layer.
- Application developers model the world in term of objects, model structures and APIs.
- Storage layer stores these data structures with data model like JSON, XML documents or tables in relational databases, or a graph model.

## Relationsal Model vs Document Model
- SQL/Relational Data Model - each relation is an unordered collections of tuples.
- NoSQL - driving forth are 
    - need for greater scalability, including very large dataset or high write throughput. 
    - desire for more dynamic and expressive data model.

Many to many relationship is more natural modeled in relational models, than document models. Most document model DBs don't have good join supports.

| Data Model | Pros | Cons |
| ------------|-----|------|
| Relational Model | Better handle joins, many-to-one or many-to-many relationship | Unnecessarily complicated application code, poor storage locality (data split across multiple tables on disks) |
| Document Model | Schema flexibiity through schema-on-read/schemaless, data locality for queries | poor supports for join
---

- Schema-on-read - Document databases sometimes are called schemaless, which is misleading. A more accurate term is schema-on-read, aka, the structure of the data is implicit, and only interpreted when data is read, in contrast with schema-on-write when schema is defined explicitly and databases enforce is on write. 
- [] Check Chapter 4 on schema evolution support

- Document and relational databases are now converging
    - Relational databases support JSON data type
    - Document databases support joins either via query languages, or drivers.

## Query Languages for Data 
- Declarative: describe what, leave the how to databases, more concise, hides implementation details, e.g. SQL. Easier to leverage CPUs' parallel execution. Preferred both in databases or web browsers.
- Imperative: requires programmer to outline the exact procedures and control flows for data retrieval, manipulations etc.

## Map-Reduce Query
- In between declarative and imperative

## Graph Data Models
- Property data model (vertex and edges) & Cypher query language
- Triple-store model (subject, predicate, object) and SPARQL query language


## Related Concepts
Client Server Communication Options: Rest vs. GraphQL vs gRPC 

- Rest is **an architecture**, easy to build, yet can created coupling between client and server, subject to both over fetching and under fetching data.

- GraphQL is **a query language** for APIs. [GraphQL vs. Rest](https://www.youtube.com/watch?v=PTfZcN20fro)

- [gRPC](https://www.youtube.com/watch?v=hVrwuMnCtok&t=212s) is a modern communication **framework**, uses HTTP2.0, and protocol buffer, for internal heterogeneous microservices. Usually choosen for convenience and performance. Born from Google. 


NoSQL Database
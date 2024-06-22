# Encoding and Evolution
- Evolvability: making application change easy (whensending data to another process that doesn't share memory, one needs to encode the data, and ensuring forward and backward compatibility )
- Problem Statement: when data is changed, application code accessing the data needs change. In large systems, changes cannot happen instantaneously (rolling upgrades or clients doesn't download new app).

## Key Cmopatibility Definition
- Backward compatibility
    - Newer code can read data that was written by older code.
- Forward compatibility
    - Older code can read data that was written by newer code. 
    - Requires older code to ignore additions made by a newer version of the code.

### Why do we need to encode/decode data
Two representations of data
- In memory (data structures like list, map, pointers, etc)
- In storage (file, DB) or over network communication (a sequence of bytes since pointers are irrelavant)

Data translation between these two representation is called encoding and decoding.

### Challenges from Language build-in encoding
- Encoding is tied to a specific programming language (cannot encode/decode across different system within the same organization)
- Decoding has security risks
- Lack versioning support
- Not efficient, high CPU costs with bloated encoding

## Standard Encoding
### JSON, XML and binary variants
JSON is more compact than XML, but binary offers little advantages as field names are still part of the encodings.

### Thrift and Protocol Buffers
- Define a field tag (numerical) 
- Come with code generation tools to produce classes implement schema in various programming languages

#### Field tag and schema evolution
- Field name changes don't impact encoding/decoding as far as field tag doesn't change.
- **Forward compatiblity** When new field is added, use a new field tag, old code can ignore the new field based on the information from datatype.
- **Backward compatibility** New field added must not be required (optioanl or repeated in protocol buffer) and have default.
- Data type change can cause precision loss (e.g. int32->int64)
- In Protocol Buffer, optional -> repeated change is both backward and forward compatible.

### Avro 
- Use schema to specify the structure of the data being encoded, no field tags
- Two Schema languages: Avro IDL and JSON
- Most compact,  encoded data doesn't contain field tags nor datatypes
- Have writer's schema and reader's schema, doesn't have to the same, only to be compatible.
- When writing data, schema often written as data file header, when reading, Avro library resolves differences from writer's and reader's schema based on its evolution rules. 
- **Backward Compatibility**: old reader's schema, newer writer's schema
- **Forward Compatiblity**: new reader's schema, old writer's schema
- Add/remove field with default value
- Allowing null in Avro involves union type: 
 `union { null, long, string } field;`
 - If avro can convert the type, changing the datatype is possible.
 - Reader schema can use alias to change field name.

 #### How does the reader access writer's schema
 - Header of Avro data file (Hadoop)
 - Schemas with versions in a DB table, schema version 
 - Sending records over network connections - schema verion is negoiated on connection setup. 

 #### Dynamic Schema Generation
 - Avro is more flexible to be used for dynamically generated schema as it doesn't involve field tags which need to be manually maintained. 
 - Avro also doesn't require code generation, which is better suited for dynamically typed languages.

## Modes of Dataflow

### Dataflow Through Databases
- Data outlives code, data in the DB can be encoded with different schema versions. 
- When update new data with old schema, additional fields could be lost. 
- Schema evolution allow the entire database to appear as if it is encoded with a single schema.
- When archiving, better to use a single schema

### Dataflow Through Services
- API exposed by a server is called a service.
- In Web, the API (of web server) consists of a standardize set of protocols (HTTP, SSL) and data formats (HTML, JSON etc). Clients of web servers also include native apps, and client-side javascript applications. 
- Services expose an application-specific API that only allows inputs and outputs that are predetermined by the business logic (applica‐ tion code) of the service.
- A key design goal of a service-oriented/microservices architecture is to make the application easier to change and maintain by making services independently deployable and evolvable. 

#### Web Service
- When HTTP is used as the underlying protocol for talking to the service, it is called a web service. 
- Rest vs SOAP are two popular approaches (Rest isn't a protocol but an architecture choice, SOAP is an XML based protocol)
- An API designed according to the principles of REST is called RESTful.


#### RPC
- RPC based web services; The RPC model tries to make a request to a remote net‐ work service look the same as calling a function or method in your programming lan‐ guage, within the same process (this abstraction is called location transparency). Challenges: 
    - Local functional calls either succeed or fail; RPC could involve network issues
    - Local calls either success, throw an exception or never return; RPC results could timeout.
    - RPC might require idempotence.
    - RPC requires encoding as data go across machines.
    - RPC latency also depends on network congestion.
    - RPC could span across different languages.
- Thrift and Avro come with RPC support included, gRPC is an RPC implementation using Protocol Buffers, Finagle also uses Thrift, and Rest.li uses JSON over HTTP.
- For evolvability, it is reasonable to assume that all the servers will be updated first, and all the clients second. Thus, you only need backward compatibility on requests, and forward compatibility on responses.

#### Version
- For RESTful APIs, common approaches are to use a version number in the URL or in the HTTP Accept header.
- For services that use API keys to identify a particular client, another option is to store a client’s requested API version on the server and to allow this version selection to be updated through a separate administrative interface 

### Message-Passing Dataflow
- asynchronous message-passing systems, that the message is not sent via a direct network connection, but goes via an intermediary called a message broker (also called a message queue or message-oriented middleware), which stores the message temporarily.
- Advantages 
 - Act as a buffer, improve system reliability
 - can redeliver, thus prevent loss
 - sender doesn't need to know the receiver's IP or port (useful for cloud deployment)
 - 1: N
 - logically decouple sender and receiver
- Message Brokers
 - one process sends a message to a named queue/topic, the broker ensures the message is delivered to one or more consumers of or subscribers. 
 - a message is just a sequence of bytes with some metadata, so you can use any encoding format. If the encoding is backward and forward compatible, then publishers and consumers can be changed independently and deployed in any order.

 #### Distributed actor frameworks
 - The actor model is a programming model for concurrency in a single process. Each actor typically represents one client or entity, it may have some local state (which is not shared with any other actor), and it communicates with other actors by sending and receiving asynchro‐ nous messages.
 - In distributed actor frameworks, this programming model is used to scale an applica‐ tion across multiple nodes. The message is transparently encoded into a byte sequence, sent over the network, and decoded on the other side.
 - A distributed actor framework essentially integrates a message broker and the actor programming model into a single framework. 


 ## Related Concepts
Client Server Communication Options: Rest vs. GraphQL vs gRPC 

- Rest is **an architecture**, easy to build, yet can created coupling between client and server, subject to both over fetching and under fetching data.

- GraphQL is **a query language** for APIs. [GraphQL vs. Rest](https://www.youtube.com/watch?v=PTfZcN20fro)

- [gRPC](https://www.youtube.com/watch?v=hVrwuMnCtok&t=212s) is a modern communication **framework**, uses HTTP2.0, and protocol buffer, for internal heterogeneous microservices. Usually choosen for convenience and performance. Born from Google. 

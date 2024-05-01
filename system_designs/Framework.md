

## Type of System Designs
- Product Design
- System Design (product with deep dive on concepts from DDIA)


## Functional Requirements
- Draw a line between covered features vs out of scope (same for non functional)

## Non functional Requirements
- availability vs consistency at discuss different parts of the system
- latency
- scalability (peak patterns)
- consider isolation and security
- fault tolerance (?)

## Core Entities
- what data persisted in the system
- exchanged via APIs
- Skeleton for high-level design

## APIs
- user id shouldn't be passed via POST payload, often assume user is authenticated, and stored in session


### Security
- user Id shouldn't be passed via POST, it is a security concerns, cannot trust data coming from clients, hence timestamp is often preferred to be generated on the server side.
# Transaction

### Why need transaction
- fault tolerance implementation for data system is very complex
- simplifying the programming model for applications accessing a database, application can ignore potential error scenarios as the database would provide the safety guarantee.

### What to consider when using transactions 
- what safety guarantee transactions provide.
    - isolation levels: read committed, snapshot isolation, and serializability.
- what costs are associated

### ACID
- safety guarantee provided by transactions
    - Atomicity, Consistency, Isolation, and Durability

#### Atomicity
- atomicity simplifies this problem: if a transaction was aborted, the application can be sure that it didn’t change anything, so it can safely be retried.
- The ability to abort a transaction on error and have all writes from that transaction discarded is the defining feature of ACID atomicity. 

#### Consistency
- an application-specific notion of the database being in a “good state.” (accounting system book balance)
- not a database property, but an application property

#### Isolation
- The classic database textbooks formalize isolation as serializability: each transaction can pretend that it is the only transaction running on the entire database.
- concurrently running transactions shouldn’t interfere with each other. For example, if one transaction makes several writes, then another transaction should see either all or none of those writes, but not some subset.

#### Durability
- once a transaction has committed successfully, any data it has written will not be forgotten, even upon fault.
- In practice, there is no one technique that can provide absolute guarantees. There are only various risk-reduction techniques, including writing to disk, replicating to remote machines, and backups-and they can and should be used together. As always, it’s wise to take any theoretical “guarantees” with a healthy grain of salt.

### Single-Object and Multi-Object Operations
- multi-object transactions require some way of determining which read and write operations belong to the same transaction.
- In relational database, this is done via client's TCP connection to DB, everything between a BEGIN TRANSACTION and a COMMIT

- single object writes
    - needed, e.g. a single large JSON object
    - atomicity can be implemented using a log for crash recovery, and isolation can be implemented using a lock on each object (allowing only one thread to access an object at any one time).
    - A transaction is usually understood as a mechanism for grouping multiple operations on multiple objects into one unit of execution.
- The need for multi-object transactions
    - database with secondary index
    - foreign key

- Challenges of retry
    - retry causing transaction executed twice
    - if error is due to overload, retry makes it worse
    - only worth retry if errors are transient
    - side affects cannot be reversed
    - client failures can cause data loss.


### Weak Isolation Levels
- databases hide concurrency issues from application developers by providing transaction isolation. 
-  serializable isolation has a performance cost.

#### Read Committed
- Two guarantees
    - no dirty reads (only read committed data)
    - no dirty writes (only overwrites committed data)

- No Dirty Reads
    - If transactions aborted, uncommitted data isn't read

- No Dirty Writes
    - read commited doesn't prevent two transactions both incrementing counters. (doesn't prevent lost updates)

- Implement read committed
    - can use a read lock, but prone to performance issues 
    - usually DB keeps both an older commited value and new value set by transactions currently holding the write lock. Only after new value is committed do transactions switch over to reading the new value.
    - use write locks to prevent dirty writes

#### Snapshot isolation and Repeatable Read
- read committed isolation can have anomaly called nonrepeatable read (read skew), causing temporary inconsistency, aka read part of the multi-objects before, and after a concurrent write transactions that's changing them.
- when temporary inconsistency cannot be tolerate
    - Backups
    - Analytics queries and integrity checks

- Snapshot isolation
    - each transaction reads from a consistent snapshot of the database—that is, the transaction sees all the data that was committed in the database at the start of the transaction.

- Implement snapshot isolation
    - MVCC (multi-version concurrency control): keeping several versions of the objects to provide isolation.

- Visibility rules for observing a consistent snapshot
    - At the start of each transaction, the database makes a list of all the other transactions that are in progress at that time. Any writes that those transactions have made are ignored, even if the transactions subsequently commit.
    - Any writes made by aborted transactions are ignored.
    - Any writes made by transactions with a later transaction ID are ignored, regardless of whether those transactions have committed.
    - All other writes are visible to the application’s queries.

- An object is visible if both of the following conditions are true:
    - At the time when the reader’s transaction started, the transaction that created the object had already committed.
    - The object is not marked for deletion, or if it is, the transaction that requested deletion had not yet committed at the time when the reader’s transaction started.

- Indexes and snapshot isolation
    - index everything and filter by transaction ID
    - Various optimization, e.g. avoid index update when different versions of the same object on the same page
    - B-trees index use copy-on-writes, and background garbage collections

- Snapshot isolation also called repeatable reads

#### Preventing Lost Updates
- automatic write operations
    - leverage database's automatic update support (exclusive lock on the object being read/written)
- explicit locking
    - application code explicit locking
- automatically detecting lost updates
    - leverage DB's lost update detection 
    - a subset of optimistic locking
- compare and set
    - leverage DB's compare and set automatic operation 
- conflict resolution and replication
    - application code resolves conflicts

#### Write Skew and Phantoms
- write skew is a generalization of lost updates
-  occur if two concurrent transactions read the same objects, and then update some of those objects  (different transactions may update different objects).
- when different transactions update the same object, you get a dirty write or lost update 

- Write skew implementations
    - use a serializable isolation level
    - the second-best option in this case is probably to explicitly lock the rows that the transaction depends on.

- Examples of write skews
    - meeting room booking systems
    - multiplayer games (multiplayer moving to the same spot)
    - claiming a username (unique constraint on username from DB helps)
    - prevent double spending

- Phantoms causing write skew
    - A SELECT query checks whether some requirement is satisfied by searching for rows that match some search condition
    - Depending on the result of the first query, the application code decides how to continue 
    - if the application decides to go ahead, it makes a write (INSERT, UPDATE, or DELETE) to the database and commits the transaction.
    - The steps may occur in a different order. 
    - Where a write in one transaction changes the result of a search query in another transaction, is called a phantom

- Materializating conflicts
    - artificially introduce a lock object when no objects to attach locks, e.g. 15 mins meeting blocks

### Serializabiity
Serializable isolation is regarded as the strongest isolation level. It guarantees though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency. 

- Serial execution
    - remove concurrency entirely, execute one transaction a time
    - RAM is cheap, active dataset can fit in memory. OLTP transactions are short
    - can potentially perform well as no need for multi-thread coordination, but can only use 1 CPU
    - encapsulating transactions stored procedures
    - partitioning
        - If there is a way of partitioning dataset so that each transaction only needs to read and write data within a single partition, then each partition can have its own transaction processing thread running independently from the others. In this case, you can give each CPU core its own partition, which allows your transaction throughput to scale linearly with the number of CPU cores 

- Two phase Locking (2PL)
    - Pessimistic concurrency control mechanism
    - In 2PL, writers don’t just block other writers; they also block readers and vice versa.
    - Snapshot isolation : readers never block writers, and writers never block readers
    - The blocking of readers and writers is implemented by a having a lock on each object in the database. The lock can either be in shared mode or in exclusive mode. 
    - transaction throughput and response times of queries are significantly worse under two-phase locking than under weak isolation, due to reduced concurrency
    - predicate locks
        - similarly to the shared/exclusive lock, but belongs to all objects that match some search condition, 
    - index-range locks

- Serializable Snapshot Isolation (SSI)
    - serializable snapshot isolation is an optimistic concurrency control tech‐ nique.
    - assumes no concurrent transactions, only commit for transactions executed serializably
    - Decisions based on an outdated premise
    - Detecting reads of a stale MVCC object version 
    - Detecting writes that affect prior reads 
    - read-only queries can run on a consistent snapshot without requiring any locks, which is very appealing for read-heavy workloads.
    - serializable snapshot isolation is not limited to the throughput of a single CPU core
    - The rate of aborts significantly affects the overall performance of SSI. 


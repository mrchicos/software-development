
# Stream Processing

A “stream” refers to data that is incrementally made available over time. We will look at event streams as a data management mechanism: the unbounded, incrementally processed counterpart to the batch data we saw in the last chapter.

## Transmitting Event Streams

### Messaging Systems
For Pub/sub model to differentiate, we look at what would happen when
- Producer is faster than consumer
    - drop messages
    - buffer messages in a queue
    - backpressure
- Node temporarily crash
    - Durability requires writing to disks and/or replication

Types of pub/sub models
- Direct messaging from producers to consumers
- Message Brokers
- Message brokers compared to databases
    - message brokers that can participate in two-phase commit
- Multiple consumers
    - load balancing
        - each message is delivered to one of the consumers, aka shared subscription
    - **Fan out**
        - each message is delivered to all consumers    
- Acknowledgements and redelivery
    - clients can acknowledge the broker upon message processing

Batch processing needs to artificially divide data into chunks of fixed duration. 

### Partitioned Logs
- Using logs for message storage
    - message within a partition is totally ordered
    - no ordering guarantee across partitions
    - within each partition, broker maintains a monotonically increasing sequence number, aka offset
    - high throughput, can write millions of messages / second 
- Logs compared to traditional messaging
    - trivally support fan out
    - for load balancing, 1 consumer is assigned to a partition, if a single message is slow to process, it can holds up the processing of subsequent messages in the partition
- Consumer offsets
    - consumer keeps track of the offset
    - consumer commits the offset back to Kafka Broker
    - Broker persists the offset in __consumer_offsets, restarted consumers can leverage this information.
- Disk Usages
    - circular buffer is used
    - regardless how long to remain message, throughput of logs remains the same
    - messaging system keeping message in memory would slow down if overflow to disk.
- When consumers cannot keep up with producers
- Replaying old messages

## Databases and Streams

### Keeping System in Sync
- Dual writes are error prone

### Change Data Capture
- Implementing change data capture
    - make the DB as the leader, and the other systems as followers
- Initial snapshot
- Log compaction
- API support for change streams

### Event Sourcing
- application logic is explicitly built on the basis of immutable events that are written to an event log.
- Deriving current state from the event log
- Commands and events
    - requests coming from the clients are commands
    - only after validation it becomes events

### State, Streams, and Immutability
- Advantages of immutable events
    - auditability
    - captures more information than current snapshots
- Deriving several views from the same event log
- Concurrency control
    - async
    - To ensure read after write consistency, transaction / distributed transaction is required

## Processing Streams

What to do with streams
- Persist in database, cache, search index or other storage system
- Email alert/notifications, dashboard, aka back to users
- Process 1 or more streams to produce 1 or more streams

Because a stream never ends, sorting doesn't apply with unbounded dataset, hence cannot use sort-merge join. Fault tolerance also different from batch processing.

### Uses of String Processing
- Complex event processing
- Stream analytics
- Maintaining materialized views
- Search on streams
- Message passing and RPC

### Reasoning about time
- Event time vs processing time
    - when using processing time to measure rates, a restart of the processor can cause event rate fluncturation
- Knowing when one is ready
    - stagger event 
- Multiple timestamps
    - event creation timestamp on device
    - client sent timestamp
    - server timestamp
- Types of windows
    - Tumbling window 
        - fixed length
    - Hopping window
        - fixed length with overlap
    - Sliding window
        - all events occured within a fixed interview, implemented as circular buffer
    - Session window
        - no fixed duration, grouped by events from same user occur closely in time.

### Stream Joins
- Stream stream join (window join)
- Stream table join (stream enrichment)
- Table-table join (materialized view maintainance)
- Time-dependence of joins

### Fault Tolerance
- microbatching and checkpointing
- Atomic commit
    - distributed transaction
- Idempotence
    - safely retry without taking effect twice
- rebuilding state after a failure
# Consistency and Consensus
One of the most important abstractions for distributed systems is consensus: that is, getting all of the nodes to agree on something.

##  Consistency Guarantees
- Eventual consistency: the inconsistency is temporary, and it eventually resolves itself
- Systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guarantees
- Consistency vs transaction isolation
    - transaction isolation: avoiding race conditions due to concurrently executing transactions
    - distributed consistency: coordinating the state of replicas in the face of delays and faults.

This chapter focuses on
- Linearizability: strongest consistency guarantee
- Ordering events in a distributed system, around causality and total ordering.
- How to atomically commit a distributed transaction, solutions for the consensus problem.

### Linearizability
- linearizability also known as atomic consistency, strong consistency, immediate consistency, or external consistency
- linearizability: make a system appear as if there were only one copy of the data, and all operations on it are atomic. With this guarantee, even though there may be multiple replicas in reality, the application does not need to worry about them.
- linearizability is a recency guarantee
- linearizability vs. serializability
    - an isolation property of transactions, where every transaction may read and write multiple objects (rows, documents, records)
    - a recency guarantee on reads and writes of a register (an individual object)

#### Relying on Linearizability
- Locking and leader election
    - single leader, coordination services like zookeeper, distributed locking
- Constraints and uniqueness guarantees
    - unique username, bank account balance, prevent double booking requires there to be a single up-to-date value
- Cross-channel timing dependencies
    - The linearizability violation was only noticed because there was an additional communication channel in the system

#### Implementing Linearizable Systems
- Single-leader replication (potentially linearizable)
- Consensus algorithms (linearizable)
- Multi-leader replication (not linearizable)
- Leaderless replication (probably not linearizable)

#### The Cost of Linearizability
- The CAP theorem 
    - when application requires linearizability, network faults cause replications unavailable.
    - when application does not require linearizability, applications can remain available upon network problems.
    -  Consistency, Availability, Partition tolerance: network faults are unavoidable, hence choose between consistency vs availability
- Linearizability and network delays
    - Linearizability is slow, weaker consistency model is faster

### Ordering Guarantees
- Ordering
    - order of writes in the replication log
    - Serializability: ensuring transactions behave as if they were executed in some sequential order
    - timestamps and clocks in distributed systems: introduce order into a disorderly world
- there are deep connections between ordering, linearizability, and consensus.

#### Ordering and Causality
- causally consistent:  a system obeys the ordering imposed by causality
- Causality imposes an ordering on events: cause comes before effect;
- The causal order is not a total order 
    - (e.g. mathematical sets are partially ordered)
    - A total order allows any two elements to be compared
    - Linearizability: a total order of operations, hence no concurrent operations in a linearizable datastore
    - Causality: partial order
- Linearizability is stronger than causal consistency
    - causal consistency is the strongest consistency without performance impact
- Capturing causal dependencies
    - when a replica processes an operation, it must ensure that all causally preceding operations have been processed
    - version vectors can be generalized to track causal dependencies across the entire database (across multiple keys, when addressing concurrent writes across replicas, version vector is used for single key) 
    - version number passed back to DB for a write <font color="lightblue">DRILL TODO!!!</font>

#### Sequence Number Ordering
- Noncausal sequence number generators
    - each node generate its own unique number, prepend a unique node identifier.
    - attach a timestamp.
    - preallocate blocks of sequence numbers.
    - above operations are more scalable than pushing all operations through a single leader that increment a counter. 
    - but the sequence numbers are not consistent with causality.
- Lamport timestamps
    - Lamport timestamp is consistent with causality.
    - The Lamport timestamp is then simply a pair of (counter, node ID). 
    - every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request. When a node receives a request or response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.
    - version vector distinguish whether two operations are concurrent or one is causally dependent on the other.
    - Lamport timestamps cannot 
- Timestamp ordering is not sufficient
    -  in order to implement something like a uniqueness constraint for usernames, it’s not sufficient to have a total ordering of operations—you also need to know when that order is finalized. 

#### Total Order Broadcast (when your total order is finalized)
- Total order broadcast safety properties
    - Reliable delivery: No messages are lost: if a message is delivered to one node, it is delivered to all nodes.
    - Totally ordered delivery: Messages are delivered to every node in the same order.
- Using total order broadcast
    - keep replications consistent
    - serializable transactions
    - locking service: every request to acquire the lock is appended as a message to the log, and all messages are sequentially numbered in the order they appear in the log. The sequence number can then serve as a fencing token, because it is monotonically increasing. In ZooKeeper, this sequence number is called zxid.
- Implementing linearizable storage using total order broadcast
    - total order broadcast is asynchronous: messages are guaranteed to be delivered relia‐ bly in a fixed order, but there is no guarantee about when a message will be delivered
    - linearizability is a recency guarantee: a read is guaranteed to see the latest value written.
    - sequential consistency (timeline consistency): write is linearazable, but not read.
- Implementing total order broadcast using linearizable storage
    - for every message you want to send through total order broadcast, you increment-and-get the linearizable integer, and then attach the value you got from the register as a sequence number to the message. You can then send the message to all nodes (resending any lost messages), and the recipients will deliver the messages consecutively by sequence number.
    - a linearizable compare-and-set (or increment-and-get) register and total order broadcast are both equivalent to consensus.

### Distributed Transactions and Consensus
Important situations for consensus: leader election and atomic commit

#### Atomic Commit and Two-Phase Commit (2PC)
- From single-node atomic commit
    - in single databse node, atomicity is often implemented by the storage engine
    - transaction commitment crucially depends on the order in which data is durably written to disk
- Introduction to two-phase commit
    - an algorithm for achieving atomic transaction commit across multiple nodes
    - 2PC uses a new component: a coordinator (transaction manager)
    - What ensures atomicity in two-phase commit
        -  When the application wants to begin a distributed transaction, it requests a transaction ID from the coordinator. This transaction ID is globally unique.
        - The application begins a single-node transaction on each of the participants, and attaches the globally unique transaction ID to the single-node transaction. All reads and writes are done in one of these single-node transactions. If anything goes wrong at this stage (for example, a node crashes or a request times out), the coordinator or any of the participants can abort.
        - When the application is ready to commit, the coordinator sends a prepare request to all participants, tagged with the global transaction ID. If any of these requests fails or times out, the coordinator sends an abort request for that transaction ID to all participants.
        - When a participant receives the prepare request, it makes sure that it can definitely commit the transaction under all circumstances. This includes writing all transaction data to disk (a crash, a power failure, or running out of disk space is not an acceptable excuse for refusing to commit later), and checking for any conflicts or constraint violations. By replying “yes” to the coordinator, the node promises to commit the transaction without error if requested. In other words, the participant surrenders the right to abort the transaction, but without actually committing it.
        - When the coordinator has received responses to all prepare requests, it makes a definitive decision on whether to commit or abort the transaction (committing only if all participants voted “yes”). The coordinator must write that decision to its transaction log on disk so that it knows which way it decided in case it subsequently crashes. This is called the commit point.
        - Once the coordinator’s decision has been written to disk, the commit or abort request is sent to all participants. If this request fails or times out, the coordinator must retry forever until it succeeds. There is no more going back: if the decision was to commit, that decision must be enforced, no matter how many retries it takes. If a participant has crashed in the meantime, the transaction will be committed when it recovers—since the participant voted “yes,” it cannot refuse to commit when it recovers.

- Coordinator failure
    - The only way 2PC can complete is by waiting for the coordinator to recover. 
- Three-phase commit
    - Two-phase commit is called a blocking atomic commit protocol due to the fact that 2PC can become stuck waiting for the coordinator to recover.
    - three-phase commit 

#### Distributed Transactions in Practice
- Distributed transactions carry a heavy performance penalty
- Exactly-once message processing
- XA transactions
- Holding locks while in doubt
    - The database cannot release those locks until the transaction commits or aborts.
    - In 2PC, a transaction must hold onto the locks throughout the time it is in doubt, blocking other reads/writes.
- Recovering from coordinator failure: open requires manual intervention
- Limitations of distributed transactions

#### Fault-Tolerant Consensus
Consensus Problem: One or more nodes may propose values, and the consensus algorithm decides on one of those values.

- Properties consensus algorithm must satisfy
    - Uniform agreement: No two nodes decide differently.
    - Integrity: No node decides twice.
    - Validity: If a node decides value v, then v was proposed by some node.
    - Termination: Every node that does not crash eventually decides some value.

- Safety properties: uniform agreement, integrity, and validity
- Liveness properties: termination (2PC doesn't satisfy termination property when coordinator fails)
    - any consensus algorithm requires at least a majority of nodes to be functioning correctly in order to assure termination
    - most implementations of consensus ensure that the safety properties—agreement, integrity, and validity—are always met, even if a majority of nodes fail or there is a severe network problem 
    - a large-scale outage can stop the system from being able to process requests, but it cannot corrupt the consensus system by causing it to make invalid decisions.

- Consensus algorithms and total order broadcast
    -  total order broadcast requires messages to be delivered exactly once, in the same order, to all nodes
    - this is equivalent to performing several rounds of consensus: in each round, nodes propose the message that they want to send next, and then decide on the next message to be delivered in the total order
    - total order broadcast is equivalent to repeated rounds of consensus (each consensus decision corresponding to one message delivery)

- Single-leader replication and consensus
- <font color='lightblue'> Epoch numbering and quorums ??? </font>
    - a weaker guarantee: the protocols define an epoch number, guarantee within each epoch, the leader is unique.
    - we have two rounds of voting: once to choose a leader, and a second time to vote on a leader’s proposal. The key insight is that the quorums for those two votes must overlap: if a vote on a proposal succeeds, at least one of the nodes that voted for it must have also participated in the most recent leader election 
- Limitations of consensus
    - Consensus systems generally rely on timeouts to detect failed nodes.

#### Membership and Coordination Services
ZooKeeper and etcd are designed to hold small amounts of data that can fit entirely in memory (although they still write to disk for durability). hat small amount of data is replicated across all the nodes using a **fault-tolerant total order broadcast algorithm***. 
- Zookeeper features
    - Linearizable atomic operations (requires consensus)
    - Total ordering of operations
    - Failure detection
    - Change notifications

- Allocating work to nodes
    - ZooKeeper runs on a fixed number of nodes (usually three or five) and performs its majority votes among those nodes while supporting a potentially large number of clients. 

- Service discovery
    - service discovery does not require consensus; DNS can do it
    - some consensus systems support read-only caching replicas.

- Membership services
    - A membership service determines which nodes are currently active and live members of a cluster
    - when couple failure detection with consensus, nodes can come to an agreement about which nodes should be considered alive or not.
    
#### Future DIGs
- Unique ID generator
- Vector clock
- SAGA


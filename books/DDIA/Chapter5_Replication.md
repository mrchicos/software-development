# Replication 

#### Why Replication
-  Reduce latency, keep data geographically close to your users
-  Increase availability (fault tolerance) - when part of the systems fail
-  Read scale

#### Replications Algorithms
- Single leader
- Multi-leader
- Leaderless

##### Trade off
- synchronous vs asynchronous replication

### Leader-based replication
- leader-based replication (also known as active/passive)
- One designated leader, writes only to leader
- When leader writes, also writes to replication log/change stream (CDC - change data capture), follower reads from the log/CDC and apply changes
- Reads can occur on both leader or follower.
- Most relational database follow this pattern, document stores like MongoDB or espresso also follow this pattern, + Kafka.

### Synchronous vs Asychronous Replication
- Synchronous guarantees follower to have up-to-date data, but blocks leader on writes.
- Asynchronous replication is more performant, but at risk for durable writes. 
- Often database either use semi-synchronous with only 1 follower synchorously write, or completely asynchronous, opting for performance over consistency.

#### Setting up new followers
- Take consistent snapshot of leader database
- copy to follower node
- Connect to leader request data change since the snapshot, the position is called log sequence number in PostgreSQL.
- process change and caught up

#### Handling Node Outages
Achieving high availability with leader-based replication.
- Follower Failure: Follower can restart and caught up based on local log of data changes.
- Leader Failure: Failover. One of the follower needs to be prompted to leader and clients need to be reconfigured. 
    - Determining leader has failed
    - choosing a new leader (a consencus problem, in Chapter 9)
    - reconfiguring the system to use new leader (request routing)

Where automated failover can go wrong
- When asynchronous replication is used, leader's unreplicated writes can got lost, violate clients' data durability expectations.
- Discard writes can cause privacy concerns due to primary keys conflicts.
- Split brain can happen.
- Hard to tune leader timeout.

These issues—node failures; unreliable networks; and trade-offs around replica consistency, durability, availability, and latency are in fact fundamental problems in distributed systems.

### Implemenation of Replication Logs
- Statement based replication 
    - cannot handle nondeterminstic function, e.g. NOW
- Write ahead log shipping (WAL)
    - tied to specific DB storage format, may prevent zero-down time upgrade  
- Logical (row-based) log replication
    - change data capture
- Trigger based application

### Problems with Replication Log
- Read scaling architecture with asynchronous replication and eventual consistency
- Read after write consistency
    - read one's own data from leader, but others from follower
    - measure replica lag, and prevent reads if lag > 1 mins
    - client remembers timestamp for most recent writes, and requests route to replica updated to that timestamp (logical timestamp)
    - Across device read after write (for same user)
- Monotonic Reads
    - one user makes several reads in sequence, they will not see time go backward
    - can hash user's id to route reads to the same replica, cannot guarantee if the replica fail.
- Consistent Prefix Read
    - if a sequence of writes happens in a certain order, then reading those writes will see them appear in the same order.
    - One solution is to make sure that any writes that are causally related to each other are written to the same partition.
- Solutions for Replication Lag
    - design a system provides a stronger guarantee than eventual consistency
    - application logics vs database transactions


### Multi-Leader Replication
- More than 1 leader accept writes, each node process writes must forward data to other nodes. 

#### Use Case
- Multi-data center operations
    - 1 leader in each data center
    - conflicts can occur
- Clients with offline operation
    - e.g. across device synchronization, each device has a local database act as leader, and there is an asynchronous multi-leader replication process (sync) between the replicas of your local app on all of your devices.
- Collaborative editing, with conflict resolutions

#### Handle Write Conflicts
- Synchronous vs asynchronous conflict detection
    - if using synchronous, then loose the point to have multi-leader
- Conflict avoidance
    - write for the same record goes to the same leader 
- Converging toward a consistent state
    - give each write a unique ID (timestamp, UUID, random number), pick the write with highest ID, and throw the rest. LWW 
    - Give each replica a unique ID, let replica with higher number overwrites lower IDs, imply data loss.
    - Merge all values changes together
    - Application solves conflict
- Custom conflict Resolution
    - on write
    - on read

#### Multi-leader replication topologies
- A replication topology describes the communication paths along which writes are propagated from one node to another.
    - all-to-all
    - star
    - circular
- casual ordering of writes


### Leaderless Replication
- In single-leader or multi-leader replication, leader determines the order in which writes should be processed. 
- Leaderless replication is also called dynamno-style
- Replication scheme ensure all data is copied to every replica.
    - Read repair, read requests will resolve differences and update stale replica on read
    - anti-entropy process, a separate process constantly looks for differences in the data between replicas and copies any missing data from one replica to another
- Leaderless replication is also suitable for multi-datacenter operations, designed to tolerate conflicting writes, latency spikes.

#### Quorums for reading and writing
The quorum condition, w + r > n, allows the system to tolerate unavailable nodes as follows:
- If w < n, we can still process writes if a node is unavailable.
- If r < n, we can still process reads if a node is unavailable.
- If w + r > n, at least one of the r replicas you read from must have seen the most recent successful write.
- If w + r < n, then likely to read stale value, trading higher availability for lower latency.

#### Slopyy Quorums and Hinted Handoff
- A sloppy quorum: writes and reads still require w and r successful responses, but those may include nodes that are not among the designated n “home” nodes for a value.
- Any writes that one node temporarily accepted on behalf of another node are sent to the appropriate “home” nodes. This is called hinted handoff. 
- an assurance for durability not a quorum

#### Detecting concurrent writes
- last write wins (LWW)
- "happens before" relationship and concurrency
- capture happens before relationship

#### Version Vector / Vector Clock
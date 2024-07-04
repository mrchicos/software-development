# Partition

#### Why partition
- scalability - different partitions can be placed on different nodes in a shared-nothing cluster, large dataset can be distributed across many disks, query loads can be distributed across many processors

#### Key problems
- how indexing of data interacts with partition
- rebalancing

## Partitioning and Replcation 
The choice of partitioning scheme is mostly independent of the choice of replication scheme

## Partition of Key-Value Data
- Some partitions have more data or queries than others, we call it skewed.
- A partition with disproportion‐ ately high load is called a hot spot.
- random assigning data to nodes avoid hot spot but cause read inefficiency, read has to query all nodes in parallel to find the data.

#### Partition by Key Range
- based on predefined key ranges
- within partitions, keys are sorted to support range scan.
- can have hotspot, keys need to be chosen carefully, e.g. cannot use timestamp as key for sensor applications, otherwise all writes from the same day goes to the same partition.

#### Partition by Hash of Key
- A 32-bit hash function generates a seemingly random number to avoid hot spot/or skewed.
- suitable hash function: doesn't need to be crytographically strong, MD5 is good. Need to generate the same hash for the same key.
- Each partition can be assigned for a range of hashed key
- Parition boundary can be evenly spaced, or chosen pseudorandomly (consistent hashing)
- Consisteny hashing / hashing partition loose the benefit of efficient range query support.
- Compromise: compound primary key consisting of several columns, only the first key is used for hashing partitions, the rest are used as concatenated index for sorting data in SSTable. Thus in twitter case, user_id is the first key, and timestamp can be used as sort key to support range query for primary key.

#### Skewed Workloads and Retrieving Hot Spots
- Most system are not able to automatically compensate for a highly skewed workload , so it is the responsibility of the application to reduce the skew. 
- Can append a random 2 digits for the hot key on write, at the expense of read (need to retrieve all partitions 1 a single key), hence better for just a few hot keys tagged. 

## Partitioning and Secondary Indexes
A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a particular value. The problem with secondary indexes is that they don’t map neatly to partitions.

- Partition secondary index by Document  
    - each partition maintains its own secondary index, covering only documents in that partition.
    - a document partition index is also called local index.
    - often secondary index are not located on the same partition, read query based on secondary index requires scatter and gather, making such query expensive. writes are efficient. 

- Partition secondary indexes by Term
    -  a global index that covers data in all partitions, it can be partitioned differently from the primary key index.
    - This is called term-partitioned
    - Partitioning by the term itself can be useful for range scans
    - partitioning on a hash of the term gives a more even distribution of load.
    - writes are slower and more complicated, may affect multiple
partitions of the index; reads are efficient.
    - writes should involves distributed transactions, in practice often async to opt for speed.

## Rebalancing Partitions
The process of moving load from one node in the cluster to another is called reba‐ lancing. Rebalancing needed when
- query throughput changes, need more CPUs
- datasize increases, need more RAM
- failover

Requirements Rebalancing shall meet after
- load (data storage, read/writes requests) shared fairly among nodes
- DB shall continue accept reads and writes during rebalancing
- No more data than necessary should be moved between nodes (fast, and minimize network and disk I/O load)

### Strategies for Balancing 
- How not to do it: hash mod N
    -  hash(key) mod N causing most of the keys to be moved, making rebalancing excessively expensive
- Fixed number of partitions
    - total number of partition doesn't change, only partition assignment to node change. 
    - only entire partitions are moved between nodes. 
    - operationally simpler
    - choosing the right number is hard, as dataset highly variable, too high causing operational overhead, too low cause outgrow
- Dynamic partitioning
    - key ranged partitions database create partitions dynamically
    - partitions can be merged or split
    - An initial set of partitions are configured pre-splitting
- Partitioning proportionally to nodes
    - in dynamic partitioning, # of partitions is proportional to the size of the dataset
    - fixed number of partition, size of each partition is proportional to dataset
    - 3rd option is to have # of partition propontial to # of nodes, aka fixed number of partition / node

### Operations: Automatic or Manual Rebalancing
Automation can be dangerous in combination with automatic failure detection. When one node is overloaded, other nodes conclude that the overloaded node is dead, and automatically rebalance the cluster to move load away from it, can cause cascading failures. 

## Request Routing
Service discovery - As partitions are rebalanced, the assignment of partitions to nodes changes. service discovery address the question of routing a key to the right node. 

Approaches
- allow clients to contact any node
- send all requests from clients to a routing tier first, the routing tier acts as a partition-aware load balancer.
- clients are partition aware, can connect directly

How does the component making the routing decision learn about changes in the assignment of partitions to nodes

- Leverage Zookeeper (coordination service) to keep track of cluster metadata   
    - ZooKeeper maintains the authoritative mapping of partitions to nodes
    - Routing tier or the partitioning-aware client, can subscribe to ZooKeeper
    - Parition ownership change will be notified by zookeepr to its subscribers
    - Can use DNS for IP resolution (as partition to node change is less frequent)

- Gossip protocols
    - more complexity on each node, but remove a routing layer

### Parallel Query Execution
- NoSQL read/write single key + scatter gather on secondary index
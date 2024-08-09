# Batch Processing

Three types of systems
- Services (online system) 
    - requests/queries, response/results
- Batch processing system (offline system)
- Stream processing system (near-real-time system)

MapReduce and several other batch processing algorithms and frameworks

## MapReduce and Distributed FileSystems
- a daemon process running on each machine, exposing a network service that allows other nodes to access files stored on that machine
- A central server called the NameNode keeps track of which file blocks are stored on which machine
- In order to tolerate machine and disk failures, file blocks are replicated on multiple machines

### Map Reduce Job Execution
- the role of the mapper is to prepare the data by putting it into a form that is suitable for sorting, and the role of the reducer is to process the data that has been sorted.
- The mapper and reducer only operate on one record at a time; the framework can handle the complexities of moving data between machines.
- The MapReduce scheduler tries to run each mapper on one of the machines that stores a replica of the input file. This principle is known as putting the computation near the data: it saves copying the input file over the network, reducing network load and increasing locality.
- The process of partitioning by reducer, sorting, and copying data partitions from mappers to reducers is known as the shuffle 
- Usually a collection of jobs are chained together to accomplish a goal

### Reduce-Side Joins and Grouping
- analysis of user activity events
- sort-merge joins
- Bringing related data together in the same place
- Group By
- Handling Skew
    - replicate data related to hot keys to several reducers

### Map-Side Joins
- reduce side joins: no need to make any assumptions of the input
- each mapper simply reads one input file block from the distributed filesystem and writes one output file to the filesystem.
    - **broadcast hash join** - one dataset for join is small enough to fit into memory
    - **partitioned hash join** - two join dataset are partitioned with the same hash
    - **map-side merge joins** - Input datasets are partitioned the same way, also sorted based on the same key.
    - **mapreduce workflows with map-side joins** - The output of a reduce-side join is partitioned and sorted by the join key, whereas the output of a map-side join is partitioned and sorted in the same way as the large input

### The output of batch workflows
- search indexes
- key value stores
- Philosophy of batch process outputs
    - treating inputs as immutable and avoiding side affects

### Comparing Hadoop to Distributed Databases
- Diversity of storage
- Diversity of processing models
- Designing for frequent faults
    - batch systems are less sensitive to faults
    - MPP usually abort the entire transaction upon single node failures

## Beyond MapReduce

### Materialization of Intermediate State
Downsize of Materialization
- A mapreduce job only starts when the preceding jobs all completed
- Mappers of redundant
- Intermediate states are temporary, replication writes cost performance

Dataflow Engine
- operators are a generalization of map and reduce
- intermediate state between operators usually kept in memory or written to local disk, which requires less I/O than writing it to HDFS
- the same processing code can run on either execution engine (e.g. Pig on MapReduce can be migrated to Spark with configuration chage)

Fault Tolerance
- checkpoints

### Graphs and Iterative Processing

### High-Level APIs and Languages
- move towards more declarative query languages
- specialization or particular domains
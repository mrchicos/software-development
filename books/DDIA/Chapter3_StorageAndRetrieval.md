# Storage and Retrieval

On the most fundamental level, a database needs to do two things: when you give it some data, it should store the data, and when you ask it again later, it should give the data back to you.

Storage engines are optimized differently between Transaction Processing databases vs Analytics databases. 

## Storage engines used in relational DBs and NoSQL DBs
- Log structured storage engine
- page-oriented storage engine

Append-only for primary data + additional metadata to index
An index is an additional structure that is derived from the primary data. Maintaining additional structures incurs overhead, especially on writes. This is an important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes.

### Hash Indexes
- In memory hashmap of key mapped to a byte offset of the value location on the data file.  
- Write appends to the file/log, and updates the hashmap.
- Log breaks into segments
- Compactions applied on segments, throwing away duplicate keys in the log, keeping only the most recent updates for each key.
- Segments are never modified after they have been written, so the merged segment is written to a new file. Merging can be done in a background thread, and switch read requests to newly merged segment once done.
- Each segment then has its own hash table index.

Some details
- File format: a binary format that encode the length of strength followed by raw string
- Deleting records: use tombstone record
- Crash recovery: index can be reconstructed from segment or index-snapshot
- Partially written records: use checksum
- Concurrency control: single writer as a common pattern

Cons:
- Hash table must fit in memory.
- Range queries are not efficient.

### SSTables and LSM-Trees
- Sequence of key-value pairs are sorted by key
- Key appear once in the merged segment.

Advantages:
- merge is more efficient as keys are sorted even if the file is larger than memory
- No longer need to keep all index in memory to find a particular key
- Suitable for Sparse index in memory

How to build the SSTables
- Write adds to memtable - In memory balanced tree data structure
- When memtable reachs threshold, write to disk as an SSTable file.
- Read is served from memtable, then disk in reverse time sequence
- Background merge process to combine segment files.
- Can keep a separate WAL to fault tolerance.

Log-Structured Merge-Tree 
- Indexing structure 
- Storage engines that are based on this principle of merging and compacting sorted files are often called LSM storage engines.

Performance Optimizations
- LSM tree algorithm can be slow when looking up keys don't exist in DB, you have to check the memtable, then the segments all the way back to the oldest. **Bloomfilter** is used to optimize such access.
- The basic idea of LSM-trees —keeping a cascade of SSTables that are merged in the background- is simple and effective. 

### B-Trees
- Most widely used indexing structures

Optimizations
- Use copy on write instead of overwriting a page with WAL
- Store on abbrevated key to pack more keys
- Leaf pages appear sequential on disk (difficult to maintain with updates)
- Leaf page can have additional pointers to left and right pages
- Fractual tree borrows Log-structured ideas


| Index | SSTable/LSM | B-Tree |
| ------------|-----|------|
| Partition | Variable sized segments, several megabytes | Fixed sized blocks or pages, traditionally 4KB in size, each page can refer to other page on disk | 
| Write | sequential, append only | Overwrite a page on disk with new data |
| In memory | memtable | No in-memory structure |
| Concurrency | single thread write | Latches used during tree-updates | 
| Crash | | Write to WAL first, as some writes involves multiple pages, crash can happen| 
| Performance | Faster to write | Faster to read | 
| Advantages | lower write amplications, compressed better, reduce fragmentations | Higher percentile performance is more predictable, a key only exists at one place, can offer stronger transactional semantics (locks can be directly attached to B-tree) |
| Downsides | Compactions can interfere with on-going reads/writes | write amplication, less compact, prone to fragmentations | 
----

### Other Indexing Structures
- Besides primary key index, it is common to have secondary indexes. The main difference is in secondary index, keys are not unique.
- Secondary indexs can either be like posting list, or appending a row identifier to make key unique, then both log-structures indexes or B-tree can be used as secondary indexes.
- Values can be stored separately from indexes in heap. Or within the index, known as a clustered index (primary key is always a clustered index).
- A compromise between a clustered index (storing all row data within the index) and a nonclustered index (storing only references to the data within the index) is known as a covering index or index with included columns, which stores some of a table’s col‐ umns within the index. 

#### Multi-column indexes 
- The most common type of multi-column index is called a concatenated index, which simply combines several fields into one key by appending one column to another.
- Multi-dimensional indexes are a more general way of querying several columns at once, which is particularly important for geospatial data. 
- Another option for geospatial data is to translate a two-dimensional location into a single number using a space-filling curve, and then to use a regular B-tree index.

#### Full-text search and fuzzy indexes
- Lucene uses a SSTable-like structure for its term dictionary.
- In LevelDB, this in-memory index is a sparse collection of some of the keys,
- Lucene, the in-memory index is a finite state automaton over the characters in the keys, similar to a trie.

#### In memory databases
- Redis, Memcached, RAMCloud etc.
- Often in-memory DB is faster because it doesn't need encoding.
- Can provides data models that are difficult to implement with disk-based indexes

### OLTP vs OLAP
- OLTP : Online transaction processing; 
- OLAP : Online analytic processing
- OLTP systems are usually expected to be highly available and to process transactions with low latency, critical to the operation of the business.
- A data warehouse, is a separate database that analysts can query without affecting OLTP operations.
- Data is extracted from OLTP databases, transformed into an analysis-friendly schema, cleaned up, and then loaded into the data warehouse. This process of getting data into the warehouse is known as Extract–Transform–Load (ETL).

#### Star schema
- Fact and dimension table

## Column-Oriented Storage
- Don’t store all the values from one row together, but store all the values from each column together instead. 
- The column-oriented storage layout relies on each column file containing the rows in the same order. 

### Sort order in Column Storage
- can use multiple sort keys.
- When compressed, the effect is strongest on the first sort key.

### Writing to Column-Oriented Storage
- Column-oriented storage, compression, and sorting all help to make those read queries faster. 
- Writes update an in-memory data structure

## Aggregation: Data Cubes and Materialized Views
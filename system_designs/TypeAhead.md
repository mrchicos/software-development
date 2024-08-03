## Problem Definition
Autocompletion in a search box


## Functional Requirement
- a dropdown based on each character user input 
- top 10 results 
- real-time updates 
- search result is proritized (trending search etc)

## Non-Functional Requirement
- 1 billion users, 100 DAU
- 20 search / day / user
- search content is up to date daily
- availability > consistency
- low latency

## Data Model
- user input
- suggestion

## APIs
GET /suggestions?q={search-term}

## High-Level Design
- applications (dispatch)
- search node (index partitioned with prefix, handling queries)
- offline updaters
- cache

## Deep Dive 
- Different ways to implement index
    - trie
    - inverted index for full-text search ( prefix -> full word)
    - predictive machine learning model
- Tries
    - store frequency list directly in nodes
    - serialize it as distributed Prefixed hash tree (distributed key-value stores), 
    - Node data <code>{
        "id": "unique_document_id",
        "term": "search_term",
        "popularity": "term_usage_frequency",
        "timestamp": "latest_usage_time"
        }</code>
    - partition
        - key (hash function)
    - replication
- Cache
- Typos
    - fuzzy matching
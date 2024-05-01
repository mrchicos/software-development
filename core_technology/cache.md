## How to keep cache data consistent with source

- read through (upon cache miss, loading from the data source then serving from cache)
- invalidate cache on write
- write through (slow down write operation to keep cache and update in sync)
- TTL
- write back/write behind (fast but risk to loss data)
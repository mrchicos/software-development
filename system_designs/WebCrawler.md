# Web Crawler

## Functional
- Given a set of seed URLs. crawl all links traversable 
- Download content of the page with timestamp/revision
- Dedup and refreshed contents

## Non-Functional
- efficiency - crawl the web < 5 days   
- scaling - 10B pages, 2MB / page
- fault tolerance to handle failures and resume crawling without losing progress. 
- politeness - don't overload the same domain

## Data Model
- pageContent
- URL metadata
- Domain metadata

## APIs

## Data Flow
- take seed URLs
- Fetch HTML
- Extract text from HTML
- Store that text in Database
- Extract URLs from text
- Repeats till all URLs have been crawled


## High Level Design
- separate fetching HTML vs parsing pages/URL extraction 

## Deep Dive
- ensure fault tolerance
    - in-memory timers at crawer retries, prone crawler failures
    - SQS vs. Kafka 
        - SOS supports retries with exponential backoffs out of box, supports dead letter queues 
        - Kafka doesn't support consumer retry out of box, needs to have a separate queue 
- ensure politeness
    - domain metadata 
        - refresh with a TTL
        - not fetching more than 1 / second
        - domain specific rate limiter
        - apply jitter
- scaling
    - parser
    - DNS
- Efficency 
    - don't crawl a URL already crawled
    - don't parse page with a duplicate content
        - redis hash (more hardware)
        - hash as secondary index 
        - bloomfilter (not a good choice, trade off with space by probability)
    - crawler traps
- Dynamic content
    - headless browser in crawlers, slower, error prones
- Monitor the health (datalog, Newrelic)
- Large Webpages
    - decisions (use content length header to limit)
- How to handle continous updates
    - URL schedulers
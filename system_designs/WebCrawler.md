# Web Crawler

## Functional
- Given a set of seed URLs. crawl all links traversable 
- Download content of the page with timestamp/revision
- Dedup and refreshed contents

## Non-Functional
- fault tolerance to handle failures and resume crawling without losing progress. 
- politeness - don't overload the same domain
- efficiency - crawl the web < 5 days   
- 10B pages

## Data Model
- pageContent
- URL

## APIs
## High Level Design

## Deep Dive
- ensure fault tolerance
- ensure politeness
- scaling
# Web Hook
Design a system that enables an application to register and manage webhooks for handling asynchronous events or notifications from other services. Essentially, it involves creating a scalable, reliable infrastructure that can receive, queue, and forward event notifications to registered client URLs based on specific events occurring in the system, much like how Stripe notifies its users of payment events. The design must account for aspects fault tolerance, observability (of # of webhooks sent to client, and failure rates), and security.

## Functional Requirements
- can register call back URLs from clients
- receive state change, queue change, send notifications to registered URLs upon defined status change 
- retry failed requests
- <font color='red'> Observability:  </font>

## Non-Functional Requirements
- Consistency, when defined state transition occur, corresponding registered URLs should be triggered.
- <font color='red'> Reliability, support retries</font>
- Scalable
- Isolation from separate clients
- Security

## Data Model
- Event
- StateTransition
- Notification

## APIs
POST /webhook/register
POST /webhook/event

## High Level Design
## Deep Dive
- Availability
    - Webhook metadata database, read replica
    - Using message queue
    - isolation 
- Scalability
    - horizontal scaring
    - horizontal sharding message queue (hotspot shard to a particular cluster)
    - cache before metedata DB
    - rate limit
- Fault Tolerance
    - persist webhook event
    - retry (retry immediately vs exponential backoff)
- Observability
    - aggregate stats from persisted webhook events
- Security
    - authentication
        - from provider to client
            - shared secret set up during webhook registration
            - use secret_token (authO token, API keys), and store it with metadata, to identify webhook request (less common)
        - client requests authenticated via API key/auth token (auth0)
    - specific IP address
    - Encrypt at rest
    - Encrypt in transit
        - payload encryption (symmetric or asymmetric)

https://pyemma.github.io/How-to-Design-Webhook/
https://www.youtube.com/watch?v=4jvV75OD620

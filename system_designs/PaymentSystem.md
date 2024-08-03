# Payment System

## Functional
- take payment from buyer and sent to PSP
- send the money to seller

## Non-Functional
- fault tolerance
- reliability
- Reconciliation process is required
- 1M transaction / day

## Data Model
- payment event
- payment order

## APIs
POST /v1/payments
{
    buyer_info
    checkout_id
    credit_card_info
    payment_orders
}

Get /v1/payments/{:id}

## High Level Design
- payment service
    - risk check, compliance check
- payment executor (can maintain a queue)
- Ledger
- Wallet (marchant balance)

## Deep Dive
- Fault tolerance / Failed payment
    - Retry
    - Idempotency (use database unique id to ensure)
- Consistency
    - idempotency to ensure only once
    - reconciliation to address issues
    - replications - trade off on scaling vs consistency

- Security
    - use HTTPS
    - encryption (at rest, in transit, access control)
    - data loss (replication)
    - DDoS

https://newsletter.pragmaticengineer.com/p/designing-a-payment-system
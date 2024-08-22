# Auction System

Some constraints:
    - each bidder can only place 1 bid for the same item, but can update the bidding price
    - bidding histories are kept

## Functional Requirement
- Seller can list item with bidding deadline
- Buyer can place bid on an item, and receiving updates
- Auction is closed when there is no higher bid for an hour
- Winner of auction receive notification and has 10 mins to make payment

## NonFunctional Requirement
- high availability
- consistency
    - live bidding: eventual consistency is acceptable 
    - determine winner: strong consistency
- latency
- scalability
    - 1B DAU, 100k auctions daily, 10% of user place 1 bid per day, assume 10:1 read:write ratio
- read >> write

## Data Model
Auction Item
Bid

## APIs
/api/search
/api/get/item_id=<id>
/api/bid?item_id=&&price=

## High Level Design


## Deep Dive
- Auction DB cache consistency

https://pyemma.github.io/How-to-design-auction-system/
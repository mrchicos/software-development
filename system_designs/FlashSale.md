# Flash Sale

E-commerce site launching a flash sale starting at 10Am for 1 signiture item, it expected the whole inventory get sold out in an hour. We only allow 1 purchase per user account.

## Functional
- Can book from a fixed inventory
- User should see current availability for an item in the inventory
- If a payment is unsuccessful, make it available for others to purchase
- Real-time booking


## Non-Functional
- high availability, scalability
    - handle sudden spike of traffic
- Handle consistency, concurrent reads, no overbooking or under bookings, reserved inventory will be returned to the system
- Fault tolerance 
- Security
    - handle script based attack

## Data Model

## APIs
POST: booking-service\book-item\{sale-id}
{
 “bookingId”: “”,
 “status”: “”, // hold, booked, deleted
 "createdtime" : "",
 "updatedtime" : "",
 “userInfo”: {
 
 },
 “bookinginfo”: {
     “itemId”: “”,
     “price”: “”
 }
}

PUT: booking-service\book-item\{sale-id}\{booking-id} 
- update the status of the booking from on hold to booked

DELETE: booking-service\book-item\{sale-id}\{booking-id} 

## High Level
- Inventory service 
- Booking
- Checkout / Payment

## Deep Dive
- Scaling
    - CDN for static content
    - inventory queue
    - NoSQL DB to scale write (MongoDB, DyanmoDB ) (partition key)
    - isolution of DB
- Payment failures
    - retry
    - idempotency
- Consistency
    - Use SQL for checkout, easier for reconciliation
    - SAGA pattern to roll back booking, and adding item back to queue 
- Multiple items
    - similar to ticket master

https://medium.com/@surfd1001/system-design-amazon-flash-sale-ed5959f53f6f
https://www.infoq.com/presentations/shopify-architecture-flash-sale/
https://microservices.io/patterns/data/saga.html
https://dev.to/willvelida/the-saga-pattern-3o7p
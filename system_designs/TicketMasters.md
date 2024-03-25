

## Functional
- search events
- view events
- book events

## Non Functional
- prioritize availability for searching & viewing, consistency for booking events
- scalable to handle peak
- low latency search
- read >> writes

## Core Entities 
- Events
- Venures
- Performers
- Tickets
- Bookings

## APIs
- get /events/details/{eventId}
Two phase booking process: choose ticket/reserve, then purchase
- post /booking/checkout
 {
    "ticketIds": string[]
 }
- post /booking/confirm
{
    "bookingId": string
    "paymentDetails":...
}
- get /events/search?keyword=...


## High Level
- view events (CURD service)
- booking (prevent double bookings)
- search 

## Potential Deep Dive
- scaling 
- high demand event bookings
- complex search, geo sharding, speed up frequently repeated search
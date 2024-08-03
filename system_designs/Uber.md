## Functional
- rider shall see estimated fair based on orig and dest
- rider can request ride, and matched in real-time
- driver can accept ride
- driver location tracked

## Non-Functional
- Low latency ride matching
- strong consistency in ride matching
- high availability & reliability
- high throughput, handle peak hour/event traffic

## Core Entities
- rider
- driver
- ride
- location

## APIs
- get /fare-estimate?orig=..&dest=?? -> Partial<Ride>

Partial<Ride>
{
    "id": number,
    "price": number,
    "eta": DateTime
}

- post /rides/request
Body {
    "rideId": string
}
- post /drivers/location/update
Body 
{
    "lat": double,
    "long": double
}

- PUT /rides/accept
Body:
{
    "rideId": string,
}
Response:
{
    "status": "success" | "error",
    "pickupLatitude": double,
    "pickupLongitude": double
}

PUT /rides/status/update
Body:
{
    "rideId": string,
    "status": "picked_up_rider" | "completed"
}


## High Level
- Ride estimate (CURD service pattern with a 3rd mapping service)
- Ride matching
- Driver accepts ( Notifications are sent via APN (Apple Push Notification) and FCM (Firebase Cloud Messaging) for iOS and Android devices, respectively )

## Deep Dive
- Frequent driver updates (batching + the geospatial database => Redis with in memory geohash + TTL)
- manage system overload from driver location updates (dynamic location updates)
- Prevent multiple rides requests to the same driver
    - application level locks
        - can cause race conditions
        - could result in hanging locks due to failures
        - coordinating locks across instances are error prones
    - database locks
        - cron jobs to clean up locks
    - distributed locks with TTL
- Peak hours no ride requests dropped
    - queue
- Further scale reduce latency & improve throughput
    - geo-sharding & read replicas

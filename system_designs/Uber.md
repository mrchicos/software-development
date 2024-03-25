## Functional
- rider shall see estimated fair based on orig and dest
- rider can request ride
- driver can accept ride
- driver location tracked

## Non-Functional
- Low latency ride matching
- strong consistency in ride matching
- high availability & reliability
- handle peak hour/event traffic

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
- Driver accepts (Notification )

## Deep Dive
- Frequent driver updates
- Prevent multiple rides requests to the same driver
- Peak hours no ride requests dropped

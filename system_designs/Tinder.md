## Functional
- CURD profile
- Recommand matches
- Swipe left/right on matches
- 1-1 messaging

## Non Functional
- High Availability, strong consistency on profile data, eventual consistency on other parts
- low latency
- privacy

## Data Model / Entities
User
Images
Match
Messages

## APIs
CURD /profile/usr_id
PUT /image/upload
Get /matches/user_id=..
POST /message/...
GET /


## Functional
- create post
- follow and unfollow
- get social feed time in reverse chronological order
- user should be able to page through their feed

## Non-Functional
- latency, availability over consistency, up to 2 mins eventual consistency
- post and view < 500 ms
- can handle massive amount of users (2B)
- unlimited amount of followers/follows

## Data Model   
    user (userId, name, email...)
    post (id, post_content, userId, timestamp)
    follow (followerId, followeeId)

## APIs
POST /post/create => postId
{
    content:
}

POST /user/[id]/follow

GET /feed/ =>  POST[]

## High Level Design
- Create Post 
    - Post DB - key value store, DynamoDB
- Friend/Follow 
    - graph DB
    - using key, value store, composite primary key (user_following: user_followeed), with user_following as partition key, with sort key (secondary index)
- View feed - follow lots of people, and lots of posts
- Scroll - use cache, and timestamp

## Deep Dive
- Follow table
    primary key (user_following: user_followeed), partition key: user_following sort key: user_followed (sort key is used to create secondary index)
- Following Large number of users
    - fan out
    - precompute feed table in cache, userId -> postId[]
    - async workers with hybrid feeds
-
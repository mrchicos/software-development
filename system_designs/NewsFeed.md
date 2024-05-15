
## Functional
## Non-Functional
## Data Model   
    user (userId, name, email...)
    post (id, post_content, userId)
    follow
## APIs

## Scaling
- Follow table
    primary key (user_following: user_followeed), partition key: user_following sort key: user_followed
- Post table
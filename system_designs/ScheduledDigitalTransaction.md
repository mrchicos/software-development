Problem Description
- Unlimited roblox
- Scheduled transfer at a certain timestamp


Functional Requirement

Non-Functional Requirement
- transaction with strong consistency 
    - idempotency 
- availability: get a pending response for scheduled transactions
- transfered should be excuted with a few mins of the scheduled time
- fault tolerance (failures shall be retried)
- 100M daily transaction
- 10 years of data stored


Data Model
 - scheduled transfer / transaction
    - uuid
    - timestamp
    - fromId
    - toId
 - wallet
    - account id
    - balance

APIs



- cron replica (at least once)
- idempotency (at most once)
- crons vs poll 


https://medium.com/@mayilb77/design-a-distributed-job-scheduler-for-millions-of-tasks-in-daily-operations-4132dc6d645f
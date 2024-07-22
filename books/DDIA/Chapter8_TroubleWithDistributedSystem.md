# The Trouble with Distributed Systems

What might go wrong in distributed system
- problems with network
- clocks and timing issues
- system models

### Faults and Partial Failures
Partial failures are nondeterministic.

### Unreliable Networks
-  a request sent to another node and don’t receive a response, it is impossible to tell why.
- Timeout is used to handle such faults.
- Network Faults in Practice
    - very common
    - error handling of network faults need to be defined and tested
- Detecting Faults
    - cases: load balancer needs to stop sending traffic to faulty node, follower can be promoted when singler leader failed
    - when something went wrong, one can get an error response anywhere from the stacks 
- Timeouts and unbounded delays
    - pick the right timeout is tricky, choose experimentally
    - network congestion and queueing
        - TCP considers a packet to be lost if it is not acknowledged within some timeout (which is calculated from observed round-trip times), and lost packets are automatically retransmitted. Although the application does not see the packet loss and retransmission, it does see the resulting delay
- Synchronous Versus Asynchronous Networks
    - trade off is between latency vs. resource utlization

### Unreliable Clocks
- often computer use NTP (network time protocol) to adjust its clock
- Monotonic Versus Time-of-Day Clocks
    - time-of-day clocks, historically had quite a coarse-grained resolution
    - monotonic clocks, suitable for measure a duration, absolute value of the clock is meaningless, the difference tells how much time elapsed. Doesn't need synchronization, hence suitable for distributed system.
- Clock Synchronization and Accuracy
    - accuracy is hard to achieve without significant investments.
- Relying on Synchronized Clocks
    - incorrect clocks easily go unnoticed
    - Timestamps for ordering events
        - Last write wins is widely used as a conflict resolution strategy in both leaderless and multi-leader replications 
        - LWW cannot distinguish between writes occurred sequentially in quick succession and writes that were truly concurrent.
        - logical clocks are based on incrementing counters, providing relative ordering of events. Physical clocks alone don't provide ordering for events.
    - Clock readings have a confidence interval [ earliest, latest ]
    - Synchronized clocks for global snapshots
        - The most common implementation of snapshot isolation requires a monotonically increasing transaction ID.
        - use time interval, and wait till interval passed before commit
- Process Pauses
    - Limiting the impact of garbage collection
    - Response time guarantees

## Knowledge, Truth, and Lies
### The Truth Is Defined by the Majority
- many distributed algorithms rely on a quorum, that is, voting among the nodes: decisions require some minimum number of votes from several nodes in order to reduce the dependence on any one particular node.
- the leader and the lock   
- fencing token - When using a lock or lease to protect access to some resource, we need to ensure that a node that is under a false belief of being “the chosen one” cannot disrupt the rest of the system. every time the lock server grants a lock or lease, it also returns a fencing token, which is a number that increases every time a lock is granted (e.g., incremented by the lock service). We can then require that every time a client sends a write request to the storage service, it must include its current fencing token.
### Byzantine Faults
### System Model and Reality
#### Timing Assumptions
- Synchronous model
- Partially synchronous model
    - a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift 
- Asynchronous model
    - an algorithm is not allowed to make any timing assumptions

#### Node failures
- Crash-stop faults
- Crash-recovery faults
- Byzantine (arbitrary) faults

#### Correctness of an algorithm
- Uniqueness: No two requests for a fencing token return the same value.
- Monotonic sequence: If request x returned token tx, and request y returned token ty, and x completed before y began, then tx < ty.
- Availability: A node that requests a fencing token and does not crash eventually receives a response.

#### Safety and liveness
- safety (unique, monotonic sequence)
- liveness (availablity)

#### Mapping system models to the real world
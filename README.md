# Distributed system note 
## Introduction
### Reasons for replication
- Main ones:
    - Increase the reliability of a system
    - Performance
- Others:
    - Geographical scaling

### Replication as scaling technique
- Scalability issues generally appear in the form of performance problems: placing copies of data close to the processes using them can improve performance through reduction of access time and thus solve scalability problems
- Scaling dilemma: 
    - Scalability problems can be alleviated by applying replication and caching, leading to improved performance 
    - Keeping all copies consistent generally requires global synchronization, which is inherently costly in terms of performance
- Scaling dilemma solution: 
    - Relax the consistency constraints. The price paid is that copies may not always be the same everywhere
    - To what extent consistency can be relaxed depends highly on:
        - The access and update patterns of the replicated data
        - The purpose for which those data are used
---
**Consistency models**: Essentially a contract between processes and the data store. It says that if processes agree to obey certain rules, the store promises to work correctly.
## Data-centric consistency models
*Data-centric consistency models aim at providing a systemwide consistent view on a data store*
### Continuous consistency
- Continuous consistency ranges:
    - Deviation in numerical values between replicas (about value)
    - Deviation in staleness between replicas (about time)
    - Deviation with respect to the ordering of update operations (about order)
- Consistency unit (conit): specifies the unit over which consistency is to be measured
    - Check *7.2. DATA-CENTRIC CONSISTENCY MODELS* for example of conit.
    - Granularity of conits trade-off:
        - Fine-grained conits: conit represents a lot of data, this may bring replicas sooner in an inconsistent state
        - Coarse-grained conits: increase total number of conits that need to be managed
- Issues with conit:
    - Need a protocol to enforce consistency
    - Developer must specify the consistency requirements

### Consistent ordering of operations
- Sequential consistency: 
    - All processes see the same interleaving of operations
    - Also, a process “sees” the writes from all processes but only through its own reads
    - So basically, sequential consistency means all processes see the same interleaving of **READ** operations
    - Metaphor: if R-R then R-R
- Causal consistency:
    - Writes that are potentially causally related must be seen by all processes in the same order. Concurrent writes may be seen in a different order on different machines
    - Metaphor: if W-R-W then R-R
- Grouping operations (mostly refer to entry consistency):
    - Use lock

### Eventual consistency
If no updates take place for a long time, all replicas will gradually become consistent, that is, have exactly the same data stored

## Client-centric consistency models
*Client-centric consistency provides guarantees for a single client concerning the consistency of accesses to a data store by that client*

**Notation**: 
- xi is version i of x, j > i doesn't mean xj is newer than xi
- Write set WS(xi): a series of write operations that took place since initialization of x
- WS(xi;xj) indicate that xj follows from xi
- WS(xi|xj) indicate that xj might not follow xi
- Li: local data store i
- Wi, Ri: write/read operation executed by process Pi
- Wi(xm;xn) / Wi(xm|xn) has the same meaning as WS one
### Monotonic reads
- If a process reads the value of a data item x, any successive read operation on x by that process will always return that same value or a more recent value
- Metaphor: if W1(x1) -> R1(x1) at L1 then W2(x1;x2) -> R1(x2) at L2 
### Monotonic writes
- A write operation by a process on a data item x is completed before any successive write operation on x by the same process
- Metaphor: if W1(x1) at L1 then W1(x1;xj) at Ln
### Read your writes
- The effect of a write operation by a process on data item x will always be seen by a successive read operation on x by the same process
- Metaphor: if W1(x1) at L1 then W2(x1;x2) -> R1(x2)
### Writes follow reads
- A write operation by a process on a data item x following a previous read operation on x by the same process is guaranteed to take place on the same or a more recent value of x that was read
- Metaphor: if R1(xi) then W2(xi;xj)

## Replica management
- A key issue for any distributed system that supports replication is to decide where, when, and by whom replicas should be placed
- Subsequently, which mechanisms to use for keeping the replicas consistent
- The placement problem itself should be split into two subproblems: that of placing replica servers, and that of placing content

### Finding the best server location
*Nowadays it's more or less a management and commercial issue than an optimization problem*

### Content replication and placement
- Permanent replicas: initial set of replicas that constitute a distributed data store
- Server-initiated replicas: copies of a data store that exist to enhance performance, and created at the initiative of the (owner of the) data store
- Client-initiated replicas: commonly known as (client) caches - a local storage facility that is used by a client to temporarily store a copy of the data it has just requested

### Content distribution
- State versus operations:
    - Propagate only a notification of an update:
        - Use little network bandwidth
        - Work best when there are many update operations compared to read operations, that is, the read-to-write ratio is relatively small
    - Transfer data from one copy to another:
        - Useful when the read-to-write ratio is relatively high
        - Transfers are often aggregated in the sense that multiple modifications are packed into a single message, thus saving communication overhead
    - Propagate the update operation to other copies:
        - Updates can often be propagated at minimal bandwidth costs, provided the size of the parameters associated with an operation are relatively small
        - More processing power may be required by each replica, especially in those cases when operations are relatively complex
- Pull versus push protocols:
    - Push-based approach (aka server-based protocols)
        - Updates are propagated to other replicas without those replicas even asking for the updates
        - Used between permanent and server-initiated replicas, but can also be used to push updates to client caches
        - Efficient when the read-to-update ratio is relatively high
        - Generally applied when strong consistency is required
        - Issue: server needs to keep track of all client caches, which may introduce a considerable overhead at the server
    - Pull-based approach (aka client-based protocols)
        - Often used by client caches
        - Efficient when the read-to-update ratio is relatively low
        - Drawback: response time increases in the case of a cache miss
    - **Lease**: hybrid form of update propagation:
        - A promise by the server that it will push updates to the client for a specified time
        - When a lease expires, the client is forced to poll the server for updates and pull in the modified data if necessary
        - An alternative is that a client requests a new lease for pushing updates when the previous lease expires
- Unicasting versus multicasting:
    - In many cases, it is cheaper to use available multicasting facilities

### Managing replicated objects
- Challenges with Replicated Objects:
    - Preventing concurrent invocations: Methods of an object should not be executed concurrently to ensure serialized access to the object’s data
    - Consistent replicated state: All replicas of an object must see invocations in the same order to ensure consistent state across replicas
- Middleware role:
    - Middleware ensures invocations on replicated objects are passed to and handled by object servers in a consistent order
    - Correct processing order must be maintained by server threads handling these requests
- Thread scheduling:
    - Multithreaded object servers must process requests in a consistent order to avoid mixing method invocations
    - Total-ordered request delivery allows concurrent processing of different objects’ invocations while maintaining order for the same object

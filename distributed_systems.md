# Distributed systems
## Why make a system distributed?
- inherently distributed 
- better reliability
- better performace 
- to solve bigger problems

#### problems
- unreliable network
- systems may crash
- nondeterministic failures

### Networking
- Physical & virtual network have tons of variations, they need to work together.
- Latency - time until message arrives
- Bandwidth - data  volume per unit time

HTTP runs on top of TCP, single HTTP request/response is broken into network packets at TCP layer, TCP combines rearrange packets into complete messages.

### RPC 
- ideally, makes RPC call to a remote fucntion look the same as  local function call
- location transperancy - system hides where resource is located
- What if
    - server crash
    - message delayed
    - messgage lost
    - what if something gone wrong.

- Service oriented architecture/ microserivces
    - tons of diff services comminicated via RPC.
    - interface defination language(IDL) -  language agnostic API specification.

# Models of distributed systems
- the 2 general problem. 
    - general 1 always attack, even if no response recieved from general 2.
    - general 1 attacks only if +ve response recieved from general 2.

- No common knowledge : the only way of knowing something is communicating it.

- the byzantine generals problem
    - more than 2 generals, messaging is reliable, but generals can be traiters.
    - if m generals are malicious, we need minimum N = 3m + 1 generals for problem to be solved. (attack to be succesuful)
    - cryptography/digital signature is helpful here.

- 2 general problems (network behaviour model)- networks can be faulty, nodes are reliable. 
- byzantine generals problem(node behaviour model) - network is reliable, but nodes be faulty
- in real life systems, network and nodes both can be faulty.

## For a distributed system consider
### network behaviour
- Reliable : message recieved if and only if sent, may be reordered.
- fair loss : messages may be lost, duplicated, reordered.
- Arbitary (active advarsary) : malicious advarsary may interfere woth messages (spoof, drop, modify, eavesdrop, replay )

- Network partition - some links may be dropping/delaying messages for extended periods of time.
- one type of behaviour can be converted in other type of behaviour.

- **arbitary + TLS ==> fair loss (retry & dedup) ==> reliable.**

### node behaviour
- Crash-stop : node crashes & stops forever.
- Crash - recovery : node crashes, losses memory & may start executing after some time.
- Bytantine - it may deviate from algorithm. 

if node is not faulty , it is correct. one node does not know if other is correct or not.

### time behaviour
- Synchronous - known upper bound of latency, nodes works at known speed.
- partial synchronous - async for some time, sync otherwise.
- asynchronous - no time guarantees, node may stop, latency can be arbitarily.

- network latency can increase due to message loss, retry, congestion, network/route reconfiguration.
- Node execution speed can be affected due to OS scheduling issue, garbage collection, pagefaults, thrasing 

## fault tolerance :
Availability - system functioning correctly.
Failure - entire system is not working
Fault - some part of system is faulty/down
Fault tolerance -  system working with some fault.
Single point of failure

### failure detection
- problem with identifying delayed response, lost message, crashed node, temp unresponsive node.
- Eventually perfect dailure detector - spurious timeout, temp false detections.

# time clocks & ordering of events
## Clocks in distributed systems
- schedulers, timeouts, retry timers, failure detections
- performace measurements, statics, profiling
- Log files & DB -  when an event occured
- limited time validity data (cache) - TTL , certificate expiration
- determine order of events accross several distribited nodes

- physical clock - elapsed seconds 
    - quarts clocks can drift due to temp, 1 ppm --> 32 s/day, most clocks have ~ 50 ppm, atomic clocks are more accurate and significantly costly
    - Leap second - to accomodate inconsistency in earth's rotation.
- digital clock

## clock synchronization
- clock drift, error gradually increases.
- clock skew - diff between 2 clocks at some point of time 
- solution : get accurate time from a server having more accurate time
    - NTP : get accurate time from atomic clock or GPS server,

- time of day clock - seconds elapsd since unix epoch (time diff can be -ve due to NTP interference)
- monotonic - time elapsed since some arbitary time - good for measuring time elapsed on single node.

## ordering o events/messages
- network delay & clock skew can make messages appear out of order.
### the happens before relation
- partial order of events 
### causality 
- when a->b a might have caused b.
- when a || b, a can not have caused b.

# Logical time & broadcast protocols
- Physical clock does not help in determining events of order even with NTP.
- Logical clocks help with this.
## Lamport clock
- helps establish total order using local timestamp and node name combination to identify events.
- (t, N) ==> (local timestamp, node id) uniqly identifies the events.
- Lamport clock does not help with identifying concurrent events. we can not tell whether a->b or a || b

## vector clock
- stores vector of timestamp for entry for every node.
- every event increments T[i] for node i.
- send vector along with every message and keep the max of local and incoming vector.

## Broadcast protocol
- one node broadcat a message , all nodes deliver it.
- assume build upon point to point network connection.
### reliable broadcast
    - FIFO 
    - causal 
    - total order 
    - FIFO total order

## Broadcast algorithms
- Make best effort broadcast reliable by retransmitting dropped messages
- enforce delivery order on top of reliable broadcast

### Eager reliable broadcasr
- the first time a node recieves a particular message, it re-broadcast it to all nodes.
- expensive O(n^2) messages

### Gossip protocol
- forward the message to 3 nodes randomly chosen.

### total order broadcast
- single leader approach
    - Leader broadcast all messages, leader crash can halt system.
- use lamport clocks.

# Replication 
- Keeping same data on multiple nodes.
- fault tolerance, scalability, availability.
- RAID in single node.

## Idempotency
- a function is idempotent if f(x) == f(f(x))
- retry semantics
    - at most once
    - at least once
    - exactly once 
- adding and removing with retry is not idempotent.
- timestamps & tombstones.
- reconcilation of replicas 
    - keep record with latest timestamp, 
- concurrent writed by diff clients
    - last writer wins
    - multi value register

## probability of faults
- A replica can be faulty due to node crash or network partition.

## consistency
- read after write consistency
    - if write fails on 1 out of 2 replicas, client might read in correct value after write.    
- Quorem 
    - write successful only if quorem agrees(2 out of 3, for 3 replicas)
- read and write quorems
    - if n replicas
    - write successful if acks by w replicas 
    - and we read from r replicas
    - so if r + w > n, reader will get latest value (read and write quorem have at least 1 common replica)
    - reads can tolerate n - r, and write can toleate n - w, unavailable replica
- Read Repair
    - client can propogate correct value with correct timestamp in case of read inconsistency

## state machine replication
- FIFO total order broadcast all updates to all replicas
- Replica delivers update message - apply it to own state.
- applying update is deterministic
- used in database replication
- related
    - serializatable transation
    - block chain, smart contracts, distribited ledgers
- Limitation - no immediate update to state, eventual consistency

# consensus
## fault tolerant total order broadcast
- total order broadcast using single leader, what if leader fails
- Manual failover - okay for planned maintanance.
- consensus to choose new leader.
- consensus - several nodes to agree on the single value    
- algos
    - Paxos/ Multipaxos 
    - Raft, viewstamped replication, Zab(zoo keeper)
- Assumes partialy synchronous, crash recovery system model
- FLP result - no deterministic consensus algo that is guranteed to terminate in crash stop system model
- failure detector to determine suspected leader crash
- on suspected crash, choose a new leader.
- prevent two leaders at same time.(split-brain)
- every node get only one vote per term.
- can gurantee unique leader per term.

# Replica consistency
## consistency
- ACID - transaction for RDBMS
- for distributed systems
    - read after write consistency

## consistency models
### distributed transactions
- Atomicity : transaction commits or aborts.
- distributed transactions - all nodes commits or aborts. if one node crashes, all needs to commit.
- atomic commitment problem.
- sublte diff from consensus.
    - consensus can tolerate crashing nodes as long as quorem satisfied.
    - distributed transaction, 1 node crash and needs to abort.

### Two phase commit(2PC) 
- have a co-ordinator.
- phase 1 - send prepare for commit, 
- phase 2 - if ok from all nodes, send request to commit.
- what if co-ordinator crashes?
- blocked till co-ordinator recovers, 
- fault tolerant 2PC algo, based on total order broadcat

## Linearizability
- how we define "consistency" in distributed replicated data?
- Linearizability : behaves like there is only single copy data and updates are atomic.
- linearizabilty != serializability
- not linerizable even using quorem consensus.
- to make it linearizable - use blind write on replica when value is outdated.
- Atomic CAS(compare and swap) + total order broadcast help achieve it.
- simple to use
- expensive - lots of message and waiting, slow

## Eventual consistency
- updates are not available instantly, nodes are consistent eventually.
- CAP theorom : system can be strongly consistent (linearizable) or available in presence of network partition.
- Replica process operation based on local state.
- strong eventual consistency
    - eventual delivery - every update will be eventually processed by all replica.
    - convergence - 2 replicas have processed same updaes, are in same state.
- causal braodcast can be used.
- concurrent updates --> conflicts need to be resolved.

^
| atomic commit
| consensus, total order broadcase, linearizable CAS
| linearizable get/set - quorem wih read repair.
| eventual consistency

# collboration software and concurrency control
## collaboration software
- calendars, google docs
### CRDT - conflict free replicated data types
- strong eventual consistency
    - eventual delivery
    - convergence    
- map CRDTs
    1. Operation based CRDT - broadcast ops
    2. state based CRDT - broadcast state
        - state merge operations - needs to be **commitative, associative, idempotent**

- text editing CRDTs
    - operational transformation

## Google Spanner
- serializable
- lineralizable 
- shards
- uses 
    - Paxos for state machine replication
    - two phased locking for linerability
    - two phased commit for atomicity   
- lock free read only transactions
- consistent snapshots
- uses MVCC, multi version concurrency control
- Physcical or Lamport clocks does not worl for spanner in achieving above properties.
- uses TrueTime, instead if timestamp, uses time range
Canary request - send request to only some servers not all, to check if eveything fails
Map Reduce - 
    Read lots of data
    Map -- Extract something you care about from each record
    shuffle & sort
    Reduce -- aggregate, summarize , filter or transform
    write the result.

- master handles the distribution
on failures re-execute the tasks.
master failures starts from last check up.

- locality optimization

Best design might be simplest, highest performance, easy to extend
ability to estimate performace of a design without actually building it
- dont design to scale infinitely. diff designs works at different scale.
- master/workers systems for mapReuce, bigTable, etc. with hot swappable master.
- tree distribution of works/tasks help with congestion, better distributions
- requests backup tasks to help with tail latency, whoever finishes first wins
- multiple smaller tasks per machines. so recovery is fast and not painful, better load balancing.
- elastic systems :  
    overcapacity - wasted resources
    undercapacity - metldown.
    auto shrink/grow
    make system resilient to overload -- diff loadbalancing, drop bells & whistles.


-- check
- join on shards ??

-- consistent hashing
R+W > n -- strong consistency

Distributed
- storage
- Computation
- messaging 
- telemetry 

# Microservices cost 
- org changes : teams have end to end responsibility, centralized self provisioning infra
- migration runs long time - double the maintance, ops, bugs, multi master data replication
- IPC is crucial for loose coupling
- automated deployment - Spinnaker
- protect database with caches  - read from cache and on cache miss update rthe cache 
- telemetry
- reliability
- cascading failures are killer

Hadoop
- map - reduce

Spark 
- scatter/gather 

Datasets, you perform transform/action, storage agnostic use HDFS, s3, parquet, 

Kafka
- streams are first class citizens

# Consistency
- Redundancy introduces Reliability.
- Consistency here means that a read request for an entity made to any of the nodes of the database should return the same data.
- Eventual consistency offers low latency at the risk of returning stale data
- Strong consistency means every replica will write data as soon as come, but reads will block, high latency
- cassandra : multi regio distributed , wide column, tunable consistency


# Durability
- durability demands replication, 
- but replication is not enough
- Durability is limited by Network Speeds
- Durability is immediate, persistent, and replicated logging.

# Sharding
- The goal of sharding is to continually structure a system to ensure data is chunked in small enough units and spread across enough nodes that data operations are not limited by resource constraints. 

## possible problems
- Node exhaustion
- Shard exhausion
- imbalanced sharding
- hot key
- small shard set
- fragmentation
- excessive paging - thrasing   
- excessive locking

- change data capture

# Events
##  Many meaning of event driven architecture
- event notification (Events or commands)
- event carried state transfer
- event sourcing  - Event logs, all events are maintained, it is source of truth, Kafka does it, it should be possible to quickly create application state from event log. 
- CQRS 

AQMP - Advanced Queue messaging protocol

Kafka & RabbitMQ
## kafka - consumers pull messages
- Event streaming platfrom, 
- Event : something has happened, with message containing description of what happened (notification + state)
    - key value pair, can be avro, json, xml, protobuff, keys are mostly integer, strint etc
    - zookeeper : distributed consensus manager
- consumers & producers are decoupled, failure of one does not affect each other
- brokers, clusters
- topics, partitions - topics are collection of related messages. it is a persistent log. ordering in sinle partition
    - partitions are replicated accross brokers, default 3
    - load balancing & semantic partition for messages across partition
- every consumer have specific offset, are stored in special topic for fault tolerance.
- consumer group - reads from same topic, single consumer is also consumer grou
- The log - where events are written, every partition has it's own offset
- dilivery :  at most once, at least once, exactly once delivery
- idempotent consumer/producers
- consumer does not destroy message
    | record
        | header
        | key
        | value
        | timestamp
- schema registry
- kafka connect : import & export data in/out of kafka
- kafka stream


## rabbitMQ - push messages to consumers
    - Fanout - all messages to everyone
    - direct - 1 to 1
    - topic - pub/sub

API gateway
- Edge
- regional
- private
- supports HTTPS & WebSocket

NoSQL
- Graph - ACID
- Columner
- key-value
- document

- in Nosql transactions should not cross aggregate boundry  

- Write Write conflict

Load Balanecing Algo
- static
    - round robin
    - sticky round robin
    - weighted round robin
    - IP/URL hash
- dynamic : take performance metric in consideration
    - Least connection 
    - Least time 

- consistent hashing
    - store data in distriburted data base
    - store object = hash(key) % N
    - auto scalling servers go and come back constantly, static hasing wont work in that case.
    - steps
        - hash object & server both on circle/ring. for any object key, next clock wise server is assigned
    - virtual nodes helps with auto scalling skews
    - with consistent hashing only portion of keys needs to be redistributed.

- LSM Trees(log structured merge trees) - Nosql fast writes
- B+ trees - fast writes SQL
- SSTables(sorted string tables) - once memtable fills, it is written as new segment of LSM tree.
- tombStone marker - like delete flag
- SSTables are compacted in lower levels, all levels make LSM Tree, data is sorted, merge is fast.
- compaction strategy affects performance.

-- Red,s
- in memory DB
- single threaded multiplexed IO
- multiplexed IO  like select, poll, epoll.

TODO - in details

- Docker 
    - contain everything an application needs in container, build & ship.
    - shared operating system, light weight(mostly due to no need for guest OS) and runs in isolated environment
    - docker file - describes build process for an image, run docker file to create image.
    - run image to run your application on docker.

- container orchestration -  k8s
- Kebernetes 
    - automating deployment, scalling, scheduling, scalling & management of containerized application.
    - manage containers according to desired state
    - deployment yaml, for desired state
    - control panel
        - scheduler
        - controller 
        - etcd - distributed kv store
        - api server
    - worker node
        - contains pods, which contains containers
        - kubelet - deamon on each node, connects with control panel
        - container runtime
        - kubeproxy
    - replica set - how many instances are desired
    - rolling updates - 
    - self healing, auto scalling, auto rollbacks, portable(cloud or datacenter)
    - complex to setup & operational, costly
    - minikube for local devlopment
    - kubectl for command line
    - minikube dashboard gives GUI dashboard
    - kubectl 
        - create -f dep.yaml - creates deployment
        - get pods - gives running pods

## Data model
- one to many relation has tree like structure, document database models it pretty well.
- many to many relation works well with relational database
- LSM tree
- SSTable
- WAL
- Btree
- LSM tree better for faster writes, Btree better for faster writes
- Btree data in single place- strong transactional gurantees
- OLTP - online transaction processing - usually some action on limited data/row
- OLAP - online analytics processing -  analytics on hugh sets of data

## encoding 
- Json, XML, CSV (human readable)
- avro, protobuf, thrift
- backward & forward compatibility

# Data flows between processes
## via database
## via service calls
- 
    ### RPC
    - makes remote call looks like local function call. it has issues cause remote call is NOT function call in local language, timeouts, encoding, retries, idempotency etc needs to be handled.
- REST is well suited for experimentation & public API
## via async message passing

# Replication
- to keep data near user
- throughput
- availability
- single leader, multileader, leader less 


API Gateway 
- seats between client & service
- provides
    - authentication & security enforcements (from identity provider)
    - logging, monitoring, analytics, billings, 
    - load balancing, circuit breaking, rate limiter (stop DDOS)
    - protocol translation & service discovery (REST to Grpc)
    - caching 

# Forward vs reverse proxy
## forward
- client to internet
    - only ip address of proxy is given, 
    - bypass browsing restriction
    - blocks access to certain sites
    - Transperant proxy - Level 4 proxy, setup at org level.
## Reverse
- internet to internal site (NGINX)
    - protects a website
    - load balancing 
    - caching static content
    - handles SSL encryption
- Edge service like cloudFare

# CDN
- content delivery network
    - keep content closer to end user
    - point of presense (POP) Edge server(node)
    - DNS & Any cast protocols to identify edge node.
    - works as reverse proxy with hugh cache of static content
    - convert content to optimized format
    - TLS handshakes are done at edge servers
    - security - DDOS protection due to large capacity
    - availability - 

# cache  
- L1-L3 cache inside computers
- page cache, inode cache managed by OS
- client side cache in browsers (expiration policy)
- CDN (static data cache)
- cache at load balancer/ API gayeway level 
- distributed cache like redis keeps key value pair in memory
- full text searches uses inverted index
- WAL, materialized view (transaction, replication log)

# Redis
- distributed key value in-memory data structure store 
- high throughput, low latency, size is limited due to in memory
- single threaded, uses IO multiplexing to wait on multiple client, Epoll is used
- supports string, hashmap, sets, list etc
- provides persistence and append only log, but not very usable(slow) , replication is used
- Usage
    - caching (redis cluster), correct TTL, thundering 
    - session store () 
    - distributed locks (mutex)
    - rate limiter
    - gaming leaderboard (sorted sets)

# Algos
- consistent hashing (cassandra), use of virtual nodes, 
- Quad tree - for indexing spatial data
- leaky bucket - rate limiting (others sliding window, token bucket)
- trie - fast lookup for strings
- bloom filter -  faster than hash maps, firm no, possibly yes
- consensus algorithms - paxos, Raft

# architectures
- MVC (layered arch)
- Event driven architectured
- micro kernal like OS, VScode, SQl server
- micro service
- monolith

# distributed system pattern
- ambassador - envoy proxy for kubernetes
- circuit breaking
- CQRS - command query responsibility segregation
- event sourcing 
- leader election (etcd zookeeper)
- pub/sub 
- sharding 
- strangler fig pattern - refactoring

GRPC 
- protobuf (binary, compact, language independent)
- uses HTTP2 streams
- gRPC web for browsers
- inter-service communication

GraphQL 
- for specific data fetch
- fetch the data just needed
- difficult to cache
- needs specific library

APIS
- REST
- SOAP
- GraphQL
- GRPS
- WebSocket
- WebHooks

## Deployments
- Big bang deployment
- rolling 
- blue/green
- canary (in a coalmine) (% of deployment to some users)

## load Balancing
- Static
    - round robin
    - sticky round robin - same user to same server
    - weighted round robin - diff priorities to diff servers
    - IP/URL hash

- Dynamic
    - least connection
    - Least time
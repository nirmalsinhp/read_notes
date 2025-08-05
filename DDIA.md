# Reliable Scalable & Maintainable Applications
## Building blocks/functionality
- Store data & Retrieve data (Database)
- Remember result of expensive operation, to speed up read (Cache)
- Search by key or filter (search index)
- Asynnchronous Processing (Stream Processing)
- Large Data crunching periodically (batch processing)

## Reliability
- continuing to work correctly even when things go wrong.
- things that go wrong are **faults**, systems which can anticipate & cope with faults are called **fault-tolerant/ resilient**.
- Fault is not a Failure(means system stop providing service), we need to stop fault becoming failure.

### Faults
- **HardWare Faults**
  - harddisk crash, CPU crash, RAM corruption, power/ network failres etc.
  - Redundancy works well for this kind of imminenet failures.
- **Software Errors**
  - hardware errors are normally not co-related, software/system errors affect every node, and have wide spread impact.
  - software/code bug, resource hogging process, slow/unresponsive process, cascading failures etc.
- **Human Errors**
  - Human errors are major cause of failures, keep good practise, abstraction, sandbox/test environments to reduce it.
  - Have better telemetry to catch issues early.

## Scalability
- Ability of a system to cope with increased load.

### Load 
- describe systems load correctly, diff for diff type os systems, like request/second, read vs write, cache hits etc.
- identify proper load factor and scale on that basis.

### Performance
- What happens when load increase
  - if resource unchanged, how performance degrades
  - if performance needs to be kept same, how much resource needs to be grow
- measures
  - Throughput - number of records to be processed/second or time taken to run job of certain size.
  - Latency - Latency not same as response time, it duration request is latent, waiting to be processed/handled.
  - response time - time between client sending a request & recieving response. not a single number, but a distribution of values that can be measured.
- response time/performance are better to be measured in percentile. p50, p95, p99(tail latency) etc. **head of line blocking.**
- horizontal & vertical scaling, there is no silver bullet, diff system have diff requirements, and solution needs to be cater to that.(reads, writes, complexity, access patterns, response time requirements. etc matters)

## Maintainability
### Operatibility
- ease of operation to run app smoothly
  - good monitoring & observability
  - easy maintance of capacity, performace, security, data systesms
  - enticipating issues & fixing them prior
  - configuration management, ease of deployment, hot swappable machines 
### Simplicity
- simple enough for new engineers to understand & contribute, 
  - overly complex systems are hard to understand, difficult to change/maintain & error prone, make system as simple as possible without reducing functionality
  - *accidental complexity* - complexity not inherent in problem domain but in implementation
  - good abstraction help making system simple.

### Evolvibility - extensibility
- ease of change, make it adaptable to new use cases & extend.
  - no systems remain unchanged for long time.
  - making systems easy & quick to change

# Data Model & Query Language
## Data Model
- is a 1 of the most important aspect of software devlopment, impacts how software is written & how we think about problem.
- Applications are built by layering one data model on top of another. 
  - Application (class/ Data structure)
    - Json / XML/ Database
      - DB engines (stored on memory, Disk, transfered via network)
        - Physical storage in SSD/HDD/RAM
  - Every layer is abstraction on another. 
- There are many different kinds of data models, and every data model embodies assumptions about how it is going to be used. Some kinds of usage are easy and some are not supported; some operations are fast and some perform badly; some data transformations feel natural and some are awkward.

### Relational vs Document
- Relational : data is organized into relations (called tables in SQL), where each relation is an unordered collection of tuples (rows in SQL).
- originates with business data processing mainly transactional processing & batch processing.
- NoSQL : Not Only SQL, alternative to SQL/RDBMS, where more flexibility, very large scale data & throughput needed.
  - Impedednce Mismatch : diff in SQL in how data is stored/scattered across multiple tables in normalized from vs how it is used in dominant OO languages as class objects/struct.
  - ORMs like sqlalchemy, hibernet, ActiveRecord helps with Boiler plate.
  - Something like resume can be better representated in Document/Json with better locality in Document DB like Mongo.
- Everything that is meaningful to humans are subject to change in future, so if that info is duplicated, that change need to be done in all places. risky & hugh overhead, removing such inconsistencies & duplication is key idea behind *normalization*.
- so One to Many relations work great & naturally in Document DB, but many to one/ many to many relations are not very natural or easy, as there is very limited support of join & mostly join needs to be done in application code.
- Hierarchical Model - similar to document DB.
- Network Model - complex data-structure based on pointer links & access paths needs to be maintained in application code.
- Schema Flexibility
  - Document - schema on read
  - SQL - schema on write 
- Locality 

## Query Language
- Declarative Vs Imparative
  - SQL - declarative, tells what to do, not how
  - Coding Lang - imparative, talls what to do & how.
- Map Reduce type querying in Mongo DB 
  - supports both imparative (pure functions map & reduce) & declarative *aggregation pipeline* ways for map reduce data query

## Graph Data Model
- Document db handles one to many relations well, Relational DB can handle few many to many relations, if there are large number of many to many relations, Graph Data model shines here.
- Data like social network, Road networks, Web can be well modeled using graph, & graph algos can be applied on the same.

### Property Graphs
- **Neo4J**
  - stored as vertex/nodes & edges
  - vertex - ID, incoming & outgoing edges, properties(Key-value)
  - Edges - ID, head/source vertex, tail/dest vertex, relation label, properties (key-value)
- Graphs can store heterogeneous data & relations as a vertex & edges, good for evolvibility.
- **Cypher QL** : Declarative Query Language for Property Grpahs
  - ``` MATCH
        (person) - [:BORN_IN] ->() - [:WITHIN*0..] -> (us:Location {name : 'united states'})
        (person) - [:LIVES_IN] ->() - [:WITHIN*0..] -> (eu:Location {name : 'Europe'})
        RETURN
          person.name
    ```
### Triple Stores
- store Graph information as Triplets
  - (subject, predicate, Object)
    - Relation / edge - (lucy, married, allen)
    - property - (lucy, age, 33)
- SparQL
      
# Storage & Retrieval
- 2 fundamental operations to store data & search & retrieve data
- log structured 
- page oriented (B-tree)
## Data-structures in DB
- **Log** : Append only file where you add new records/data. (fast writes & slow reads)
- **Index** : Additional data structure that stores metadata & helps with speeding up reads (extra overheads on writes).
  - Hash Index : simple hash map like structure. 
    - for log based engine like Riak this technique is used, hash_map is stored in memory as a key- value store & value is mostly offset in file segment. 
    - Fast due to append only nature, when files/segment gets big they are merged & compacted, keeping only latest value for key and removing older ones.
    - Aspects to consider
      - File Format, deletion of record, crash recovery, concurrency, partially written records.
      - Limitation - if number of keys are very big, keeping everything in memory do not work well. Range queries very slow in hash based index.
### SSTable (Sorted String Table)
- key-value pair in segments are stored in sorted manner based on keys. 
- Optimized for writes.
- used in key-value NoSQL stored like LevelDB, RocksDB, Cassandra
  - + Merge is very quick based on merge sort, Range Queries, No need to store all keys in memory.
- key-value is stored in in-memory sorted data structure like red-black tree which is called ***memtable***. 
  - When memtable gets big, it is written on disk as SSTable, every key is first searched in memtable & than most recent SSTable.
  - multiple SSTable are merged periodically(based on size) in Background.
  - every write can also be written to a file as they come for recovery of memtable in case of crash,
- Database engine which uses SSTable technique, are called **LogStructured Message(LSM) Tree**.
  - Searching for a non-existent key can be very costly. Bloom Filter can be used to identify if a key does not exist. Summary Table for each level to aide in fast search
  - merge compaction can be size tiered or level tiered.

### B-Tree
- B-tree also stored data as sorted pair of key-value, but stored in very different manner than LSM Tree.
- Optimized for Reads.
- Data is stored in fixed size (typical 4KB) Pages, every page contains reference to keys in multiple range.
- every page can have reference to multiple child pages - called **Branching Factor**.
- Leaf pages stored key & Value. Modification (Add/Update/Delete) is done in leaf level & page is written to disk.
- **Write Ahead Log** : modification is written to immutable append only log before actually applying it to tree, used in recovery after crash.
- in-place writes/modification needs locking mechanism for concurrency (latches).

- **Write Amplification** : number of times record/data is written/ReWritten in Disk for single write (i.e in WAL or SSTable compaction  Merge)
- Primary Index : for searching specific keys, are unique.
- Secondary index : for sorting & Joins. 
- Clustered Index : indexed row is directly stored in index. 
- Multi-dimension Index
  - Concanated Index : concanate multiple columns & use as keys.
  - for GeoGraphic data this does not work well, R-Tree is used for this kind of index.
- Full Text search & Fuzzy index : Search words with some edit distance from orignal word/key.  Trie like data structure in used in memory for index. (Lecene, Solr, Elastisearch).
  
### InMemory Indexs
- Like MemCached & Redis : where entire dataset is kept in memory, advantages over DBs when data is stored in Disk. for Recovery data can be stored to replica, or WAL.
- Major advantage is that data does not need to be serialized for to be written in Disk & Redis like product can support/store data structures like priority queue & sorted set in DB/Cache.

## Transaction Processing or Analytics
- OLTP
- OLAP
- 
| **Property**         | **Transaction processing systems (OLTP)**         | **Analytic systems (OLAP)**            |
| -------------------- | ------------------------------------------------- | -------------------------------------- |
| Main read pattern    | Small number of records per query, fetched by key | Aggregate over large number of records |
| Main write pattern   | Random-access, low-latency writes from user input | Bulk import (ETL) or event stream      |
| Primarily used by    | End user/customer, via web application            | Internal analyst, for decision support |
| What data represents | Latest state of data (current point in time)      | History of events that happened        |
| Data size            | GB- TBs                                           | TBs- PBs                               |

- large orgs can have various OLTP systems, running large analytics queries on the same system can affect performance. so they keep seperate DBs for Analytics/report purpose called **Data WareHouse**.
- ETL (Extract Load Transform) : Extract data from various OLTP systems, clean & transform it to be more usable for OLAP and load in Data WareHouse.
  
### Star/SnowFlake Schema for Analytics
- Star schema where one table is fact table which stores an event & contain foreign_key from dimension table which contains who, where, why, how, when of the event.

### Column Oriented Storage
- OLTP databases store(row-oriented) row data in contiguos memory, so if you need very few columns for the analytics query, it still needs to read all the columns in a row, it can take long time.
- OLAP (column -oriented) DBs store data for every column in seperate file in contiguous storage. it makes analytics query needing only some of the columns fast.
- Column compression 
  - bit encoding & run Length encoding
  - sorting column can further help with it, can store multiple sort orders.

# Encoding & Evolution
- Applications change over time, new fields to be added, new feature to be added, data needs to representated in different format, requirement changes etc. 
- for a large applications rolling updates help with reliability.
- Applications need to support 
  - BackWord compatibility : new code read data written by old code.
  - forward compatibility : old code read data written by new code.

## Formats for Encoding data
- Data exist in 2 format. 
  - in-memory in form of data-structures, optimized for efficient CPU access.
  - Write to file/send over network as some kind of byte sequence.
- encoding(serialization/marshalling) = in-memory -> byte sequence 
- decoding(deserialization/unmarshalling) = byte sequence -> in-memory
  
### Language specific formats
- Pickle for Python, Serializable for JAVA etc.
  - Bad performance, language dependent, no versioning or backward/forward compatibility.
- Avoid it in most cases.

### Text formats
- JSON, XML, CSV are prevalent formats with usefulness, dislikes, strnghts & Issues
  - ambuguity/issues around representing numbers or string with numbers only
  - support for binary strings/sequence is very minimal, base64 encoding is used to overcome, but costly.
  - no schema support generally.

### Binary Encoding
- XML & JSON are widely used but not very efficient & verbose in general, it affects performance when data goes into TB, many binary format around JSON & XML are introduced like webXML, BSON, Bison etc. not as widely used as XML/JSON.
- Thrift & Protocol buffers : Developed at FB & Google, Binary formats with auto-generated cross-language stubs. 
  - way more compact than Json/XML.
  - provides IDL for schema which help stub generation.

- Sample
  - ```{
    "userName": "Martin",
    "favoriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
    }```

![Thrift](/Images/thrift_compact.png)

![ProtoBuff](/Images/proto_buff.png)

- both uses field tag with length instead of field names in encoding.
  - it helps with schema evolution
  - Backward compatibility can be maintained as new code can read old data
  - forward compatibility maintained as old code can simply ignore new data
- changing data type of a fields breaks compatibility.

- Avro : from Hadoop, does not use tag numbers for field, diff human readble & binary json schema. support schema evolution using keeping reader & writer scehama & resolve conflicts when reading.

## Model of DataFlow
- how data flow from one system being encoded, flown to another system & decoded.

### Dataflow through services
- one process writes data to db, and another process or same process at later time reads it.
- schema evolution also needs to be handled in this case, be aware of it, and handle accordingly.
- diff version of data exists at different time, use Avro like format to support evolution,
- Data Outlives the code.

### Data flow through services
- client - server sharing data via API.
- client can typically be web-browser, but are many other clients also like another service in service oriented architecture/MicorServices, mobile application/desktop application etc.
- when HTTP is used as the underlying protocol for talking to the service it is called web service.
- REST : simple data model, use urls for resource identification, based on HTTP protocol & uses HTTP feature for cache control, authentication, content negotiation. very popular, RESTFul services.
- SOAP : xml based protocol for network API calls, WSDL, complex, not human readble, vendor specific. needs proper tooling to work efficiently.
- OpenAPI specification & Swagger like framework made working with REST easy.
- RPC : remote procedure call, RPC method tries to make a request to remote network service look the same as calling a function locally in your programming language (location transparency).
  - historically many frameworks like EJB, Java RMI,  DCOM, CORBA have come with their own problems.
  - network call is fundamentally diff from function call, making it look like local call is flawed notion,
  - network delay, slow reponse, idempotency issues, how to handle retries, diff data type interpreation in diff language in RPC are several issues with RPC.
- despite it's disadvantages, RPC framework with Thrift, gRPC are gaining popularity, mostly used for internal service communication, not very useful for public APIs, compatibility support provided by the protocol used like Avro, protobuf, thrift.

### message passing
- passing messages in different encoded formats via message queues. they have middle ware called broker and messages are sent via named message queues / topic, many consumer/subscriber can read from same message/topic.
- asynchronous, one way communication, handles service crash by storing the message locally templorarily.
- decouples sender/reciever.
- Kafka, TIBCO, RabbitMQ, ActiveMQ etc

- **distributed actor model** as a way of processing every client is an actor with some local state & communicating via message queues.
- AKKA, orleans, erlang OTP

# **Distributed Data**
- *For a successful technology , reality must take precedence over public relations, for nature cannot be fooled.*
- Data storage & retrieval works well in single system, but when data is distributed accross machines, it is at different level of abstraction & complexity. but why distribute the data
  - Scalability
  - Fault tolerance/ high availability
  - Latency
- Scaling 
  - Vertical Scaling : 
    - shared memory arch: many CPU, RAM, disks works as a single machine on single os, works well till a limit, cost grows faster than linear.
    - shared disk arch: many CPU, RAM distributed but sharing data with single disk, contention & locking makes it usefulness limited.
  - Shared Nothing Arch / horizontal scalling : every node with disk, cpu & ram is independent. co-ordinatio/communication is at software level over network. Adds complexity.
- Replication : same data replicated over multiple nodes for redundancy
- Partition : data is partitioned on seperate nodes. (Sharding)

![dist_data](Images/replication_sharding.png)

# Replication
- *The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at or repair.*
- Keep same data on multiple machines connected via network.
- Replication to
  - reduce end user latency by keeping data in geographic proximity
  - availability : system works even if some machine is down
  - read replica : increase read throughtput
- All the difficulty lies in hadnling changes to replicated data, 3 algos to handle replication of changes.
  - **single-leader, multi-leader, leaderless**

## Leaders & Followers
- most common replication solution is leader-based replication(master-slave, active-passive).
- write goes to leader replica & changes are sent to followers via change stream/replication log. replica update it's local data
- client can read from leader/follower replica.

![Leader-based replication](Images/leader_follower.png)

### sychronous vs asynchronous replication
- synchronous : leader waits for follower replica's confirmation of write before returning success to client. highly consistent, but failure of a single node leader/follower can halt a system entirely.
- asynchronous : leader sends the message, do not wait. eventual consistent, but data can get lost in case of leader crashed.
- in most cases one replica is synchronous & others are async.
  
### setting up new followers
- can not simply copy disk data from 1 node to new one, data is constantly written, can not lock writes as it will hinder availability
- General steps
    1. take snapshot in time of leader
    2. copy the snapshot to new node & apply, 
    3. once follower is up, sync all the data after snapshot (log sequqnce number) to follwer, and continue as normal.

### Handle outages
- follwer outage : replica keeps a log of data changes on disk, if it crashes, it reads the data from disk & sync with leader from the last transaction/change on disk, continue as usual afterwards.
- Leader ourage / Failover:
  - can be handled automatically or manually.
  - determine that leader has failed : mostly based on timeout, tricky to get right
  - new leader selection : consensus algos, leader with latest data can be chosen
  - reconfigure system with writes to new leader, if old leader comes back make sure it become follower.
  - Issues:
    - diff between data in old leader & new leader, old leader's writes are usually discarded. 
    - even dangerous if database is synced with outside system like cache/redis
    - split brain : new & old both thinks they are leader
    - what should be the right timeout?

## Replication Lag Implementation
- statement based : every statement is replicated, many issues with non-deterministic functions like RAND, NOW, auto increment & statement with triggers & stored proc.
- Write-Ahead Log : write append only log to disk & also send it to followers. issues with version compatibility, as logs are dependent on underlying storage engine.
- row level(logical) reasoning : changes written as granularity of row, logical independent of physical representation & storage engine. called **change data capture**.  easier to work with data warehouse & external systems also
- trigger based : replication based on some event happened, and written to sperate table, followers can read it. flexible, needs extra application level code. high overhead than built in replication.

## Replication Lag
- In replication system with many read replicas to support high read load, sync replication does not work as single failure can halt everything, so async replication is used. in this case many of the replica can be behind leader, and client reading data from that replica can be reading stale data. 
- if for some time leader is locked and no write is done, replica will eventually catch up & be consistent with leader, that's why it is called **eventual consistency**. replication lag can vary from few milliseconds to minutes. there are many forms of eventual consistncy like.

### read after write
- In some system client are shown the data they wrote/updated, so reading from stale replica can be an issue. To handle this case user/client can be shown data from leader, but not everything can be read from leader. 
- various ways based on type, time , location of modification can be identified what data needs to be read from leader/follower.

### Monotonic reads
- if client reads from random replica everytime, it can happen that it reads from upto date replica first, and read from stale replica afterwards, for the client it may seem **time is moving backwards**.
- need to tackle this anamoly with stronger than eventual consistency. user should not get feeling of time-travell, can tackle with user reading from same replica. and re-route to other in case of failure/crash for particular replica.
  
### consistent prefix reads
- To make sense of events happening, maintaining causality is very important.
- consistent prefix reads gurantees that related writes are read in same order as writes. maintain happens-before relationship.

- for single node systems, **transaction** exists to handle all above scenarios/anamoly, does not works very well with distributed system, alternatives are there.
  
## Multi-leader replication
- master/master, active/active setup. in sigle leader replication, if leader goes down, writes can not procceed, it is single point of failure. 
- setup where there are multiple leaders & write can heppen at any of the leaders. leaders are synced with other leaders via replication.
- multi-datacenter multi-leader replication
  - multiple data center distributed geographically can have multi-leader setup, a leader in each data center, data is synced via public internet between data center & leaders. 
  - performance : user do not need to make geographically distant write, can write to nearest leader.
  - tolerance of datacenter outage
  - tolerance of network outage : DC can work independetly.
  - dangerous, retrofitted in DBs, write conflicts among DCs tricky to handle. avoid if possible.

![multieader](/Images/multileader.png)

- clients with offline clients 
  - similar to calendar sync, each device work as independent data-center, async replication at random times, device contains local data
- Collaborative editing
  - editing like google docs, again conflict resolution is at core of the problem.
  - locking a segment/region can be plausible solution, will make everything slow.
  - work on smallest unit of data, like keystorke & avoid locking.

### Write conflicts
- Biggest issue with multi-leader replication setup is write conflict. 2 users can update the same record from & submit to 2 different leaders, at both ends write is successful, but conflict is detected at later time when replication across leaders is done.
- sync vs async conflict detection : we can write only after replication is done, but it defeats the purpose of multileader replication
- avoid conflict - same user always make update the their own data in same datacenter, problematic in case of DC failure or user changes the location.
- converging towards a consistent state
  - there is no specified order of writes in multi-leader replication, and all the writes should eventually be consistent, after replication is done data is same in all location.
  - unique Id to each write & highest ID wins, Last write wins, prone to data loss
  - give Ids to replica, higher id replica wins, prone to data loss 
  - resolve at software level at later time.
- custom conflict resolition
  - custom to the applicatio. based on software code
  - on write : conflict handler is invoked in case of conflict detected in replicaton log.
  - on read : user is prompted to select appropriate record from all conflicting values.
- Automatic conflict resolution
  - **CRDT** : conflict-free replicated data type, data structures concurrently modified by several users & resolves conflict in some internal sensible ways
  - mergable persistent data structure : 3-way merge function,
  - operation transformation : used in google docs for collaborative editing,

### Multileader topology
- many different topologies exist.
- star, circular, all to all
![topology](Images/multileader_topo.png)
- star & circular tolology can get hung if one of the leader is down.
- endless replication needs to be stopped, mostly id of the leader is attached with each forward & replication log/CDC with it's own id is ignored to stop endless loop.
- all to all replication does not have endless loop issue, but different leader can recieve data at different time, and maintaining causality of write can be an issue.
- insert is recieved at some leader after insert, due to network issue.

## Leader-less replication
- client can write to any of the replica, also called dynamo-style(not dynameo DB). cassandra, Riak, Voldemort.
- user can also read from all the replicas.

![leaderless](Images/leaderless_repl.png)
### Node outage writes
- if one of the node is down,  write can be successful based on quorum(i.e 2 out of 3). 
- read requests are also send to all nodes in parrallel & stale data can be repaired.
- read repair : if client recieved stale data from any node, latest data is written back.
- anti-antropy : background process which write latest data to nodes with stale data periodically.

### Quorum
- More generally, if there are n replicas, every write must be confirmed by w nodes to be considered successful, and we must query at least r nodes for each read. (In our example, n = 3, w = 2, r = 2.) As long as **w + r > n**, we expect to get an up-to-date value when reading, because at least one of the r nodes we’re reading from must be up to date. Reads and writes that obey these r and w values are called quorum reads and writes.  You can think of r and w as the minimum number of votes required for the read or write to be valid.
- Normally, reads and writes are always sent to all n replicas in parallel. The parameters w and r determine how many nodes we wait for—i.e., how many of the n nodes need to report success before we consider the read or write to be successful.

![quorum](Images/quorum_3.png)

- even with quorums there are many cases which can lead to in-consistency
  - sloppy quorum & hinted handoffs
    - in case of network interuptions, nodes are alive but quorum can not be reached. for cluster with nodes significantly greater than n, client may be connected to nodes, but not the one required for quorum. 2 choices
      - fail all ops
      - write on available nodes (not home node) - sloppy quorum
    - writes are then synced to home nodes when network is repaired (hinted handoffs)
  - concurrent writes on 2 nodes, due to clock skew can not depend on last write wins to identify latest.
  - read & write can occur simultaneously for same data.
  - read might fail on some nodes & quorum id not meet, writes on successful nodes is not reverted.
  - if node with latest data fails, & recovered from older data node, you may lost writes.
- **Monitoring Staleness** : very tricky to monitor the state of the system, in leader based replication you can identify staleness based on cursor in replication log, in leader vs follower. in leader less application without replication log and writes going everywhere, there is no standard way to measure staleness.
- ***Eventual consistency is a deliberately vague guarantee, but for operability it’s important to be able to quantify “eventual.”***

### Detecting concurrent writes
- many client are able to write concurrently in leader less setup, but also due to read-repair & hinted handoffs it may look like concurrent writes.
- **concurrent Writes** : does not necessarily ovelap in time, if 2 writes are un-aware of each other, they are concurrent.
- **possible solutions:**
- Last write wins : most recent write is the final, older are overwritten, due to skew in distributed clocks it is difficult to identify most recent, used in cassandra.
- Causally dependent / happens before relationship.
  - 2 ops A & B can happen
    - A before B, B before A, concurrent 
- to capture happens before relation ship, version is used with each write, and each write reads before writing, merge the data from previous reads. and keeps the latest data. **tombstone** is used to handle deletion of data in some of the writes.
- **version vector** : The collection of version vector from all the replicas is called version vector.

# Partitioning
- Replication : store same data in multiple nodes
- Paritition : for very large datasets & query throughput, that may not be sufficient, data is broken in to paritition. Normally, each data *belongs* in only one partition, used for scalability, each partition works as it's own database.
- usually, parition in combined with replication, each data belongs to one partition, but may be stored on diff nodes for fault tolerance.
![partition](Images/partition_rep.png)

## parition of key- value data
- out goal should be to distribute query & data evenly across nodes, node with dispropotionally high node is called *hot spot*, it can be a bottleneck.
- in simple key-value setup we can partition by key.
- **partition by key-range**
  - data is partitioned in key-ranges, like A-G, H-L etc. prone to hot-spot if partition not decided carefully. 
  - keys are sorted, so scan are usually fast & efficient.
- **paritition by hashing**
  - keys are paritioned by hasshing the keys, due to hash keys are uniformly divided among nodes.
  - not very good with range queries as not sorted.
  - **consistent hashing**

### secondary idexes with partition
- secondary indexes are useful for faster searches & range queries, maintaining them in partitioned db is difficult.
- paritioning seconday indexes by
- document id
  - seondary indexes are also paritioned based on range, local index : specific to data in that parition only.
  - reading based on this needs to be be done carefully, fetch data from all parition. it is called scatter/gather approach
- term
  - global index, also paritioned, differently from primary index. called term index as term we are looking for identifies the parition. similar to index in text based search.
  - reads are fast, but writes get slow & complicated(update to 1 document affect multiple partition). update to secondary index is async, gets few milliseconds.

## Rebalancing partition
- Rebalancing is needed when data size, query throughput increases or machine is down & other machine needs to take over.
- the process of moving load from one node to another is called **Rebalancing**.
- For Rebalancing, 
  - data should be shared fairly afterwards, db should contiue while rebalancing, and only necessary amount of data should be transfered.(costly IO)

### Strategies:
- **hash % N** : do not do it, if node added or removed too many keys are moved, as hash % N, will change for large number of keys.
- **Fixed number of partition** : number of partition is fixed at time of creation & can not increase afterwards, max node = max partitions, when rebalancing happends entire partition is moved to another node, not much key movement is done. 
  - partition size should be such that moving them is not an overhead nor it is too large that moving them take eons. voldemort, riak, elastisearch.
- **Dynamic partition** : number of partitions are changes as per size of data, if data grows new partition is created & data is splitted, if data shrinks, partition can be merged into single.
- **partition propotional to node** : normally partition is propotion to data, can be also done propotional to number of nodes, initially, all nodes have some set of partition, once new node is added or removed, partition is split or merge accordingly, to keep consistent partition/node.
- Fully automatic or manual rebalncing, should be semi automatic, allow automatic suggestion for partition, still keep human to commit.
![req routing](images/req_routing_part.png)

## Request Routing
- how to identify which node to hit for specific key, special case of ***service discovery***. 
  - round robin - request to any node & it will handle or pass the request to appropriate node.
  - partition aware load balancer - client hit one node/gateway, it forward to correct node.
  - client decides - client has data for key-parition-node path, it makes request directly to node.
- how decision making entity knows about rebalancing/moving of partition.
- Many distributed data systems rely on a separate coordination service such as **ZooKeeper** to keep track of this cluster metadata. like kafla, HBase.
- Cassandra & Riak uses *gossip protocol*, like option1 above, push complexity to DB design, but do not need to depend upon zookeeper like external services. 
- **Massively parrallel processing** vastly diff than nosql partitioning.

# Transactions
- hardware/software crash, application crash, partial reads, clients overwrite writes, network failure, race conditions are some of the issues/faults which arise in data system.
- Transactions has been the machanism to simply this issues.
- Transaction is a way for an applicatoin to group several read & writes as single logical unit. all read/writes in transaction is executed as 1 operation. 
- it succeeds(commit) or fails(abort/rollback). simplifies application programming model with **Safey gurantees**.
  

## ACID
- **Atomicity** : Atomic means something which can not be broken down further in smalller part, in multithreading/concurrent setup it means diff than data system. 
- it describes what happens it client wants to make serveral writes, but fault occurs while write is going on. so abort the transaction, once transaction is aborted application code/client can be sure no writes has been done & safe to retry.
- **Consistency** : highly overloaded term, for data systems it means data invariants are kept after transactions. can not stop application from writing junk data.
- **isolation** : several client writing at same time can raise concurrency issue like race condition.
  - isolation means concurrently executing transaction does not effect each other. 
  - **Serializability** : every transaction can pretend it is the only transaction running in DB.
- - **Durability** : means once transaction is complete, data is written, will not be gone even in case of nertwork, hardware failures.

![lost update](images/lost%20updates.png)
## single object & multi-object operations.
- for multiple writes in single operations, DBs provide below gurantees
  - Atomicity : all or nothing, no partial failures.
  - Isolation : concurrent transaction does not interfere with each other.
- Single object writes
  - partial write failures, partial reads, corripted data due to power failures issues can arise, so atomicity & isolation also needed for single write ops.
  - Atomicity - can be implemented using log for crash recovery. 
  - isolation - using lock for each object.
  - Atomic increment, compare & set operatins are also provided by DBs.
- need to multiobject transactions
  - some apps can get-away with single object transactions. however for many cases multiple write needs to be coordinated.
    - foreign ket updates, writing to denomalized data in SQL/NoSQL DBs, update secondary indexes on write, all these need multi-object transactions.
    - without it error handling can get messy & overly complicated.
- key feature of transaction is, it can be aborted & safely re-tried if an error occurs. 
- some leaderless replicated nosql dbs, do not rollback the changes in case of error occurs, do changes on best effort basis, those are hard to handle in case of errors.
- even with transactions error handling can be tricky in some cases,
  - like error due to overload, client process crash due to error in transaction, there is side -effect to transaction operations.

## weak isolation
- If two transactions don’t touch the same data, they can safely be run in parallel, because neither depends on the other. Concurrency issues (race conditions) only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.
- For that reason, databases have long tried to hide concurrency issues from application developers by providing transaction isolation. In theory, isolation should make your life easier by letting you pretend that no concurrency is happening: serializable isolation means that the database guarantees that transactions have the same effect as if they ran serially (i.e., one at a time, without any concurrency). High performance price, DBs provide weaker isolation level than serializable.

### Read Committed
- Gurantees
- No dirty reads : When reading from the database, you will only see data that has been committed.
- No dirty writes : When writing to the database, you will only overwrite data that has been committed.
- **Dirty Read** : if another transaction can read uncommited writes.
  - if transaction changes multiple data & dirty read is allowed, it gets really confusing, what if transaction needs to be aborted & write to be rollbacked, wrong decision taken on partial data.
- **Dirty Write** : if 2 transactions writes same data, usually later writes overwrite early one, if later transaction overwrites the early write which was un-commited, it is called **dirty write**.
- Implementation :
  - Default level in many DBs,
  - dirty writes are prevented using row-level or document level locks, no transaction can take lock while other is writing to the row/documemnt.
  - to prevent dirty reads, same locking machanism can be employed, but it is very costly, many reads can blocked sue to some transaction is writing. it is prevented by maintaining 2 values, any reads are given commited value, and new value being written by transaction which is locked, new value updated after transaction is committed.

### Snapshot isolation & Repeatable read
- with read committed isolation, read skew can happen as per below.
  - if user reads some of the data before transaction & other partial read after some transaction which changes the data, user can end up with inconsistent state temporarily. it is called read skew an example of **called non- repeatable read**.
  - works fine for some cases,as reading again gives the correct data, but not for some of the cases like
    - Backup : taking backup takes hours normally, if writes is happening while backup, we can end up with different version of data & applyting that in-consistent backup can be disasterous.
    - Analytics query & integrity checks : queries or checks scanning large part of the database can return non-sensical results if they read databases at different point in time in single query.
- **Snapshot isolation** is the most common solution to this problem. The idea is that each transaction reads from a **consistent snapshot** of the database—that is, the transaction sees all the data that was committed in the database at the start of the transaction. Even if the data is subsequently changed by another transaction, each transaction sees only the old data from that particular point in time.
- Implementation 
  - write locks work same as read commited. reads do not require any locks.
  - *key principal is readers never block writers, writers never block readers*
  - generalization of dirty reads, The database must potentially keep several different committed versions of an object, because various in-progress transactions may need to see the state of the database at different points in time. Because it maintains several versions of an object side by side, this technique is known as **multi-version concurrency control (MVCC).**
![mvcc](images/mvcc.png)
- visibility rules
  - writes from in-progress transaction at start of particular transaction, from aborted transaction, transaction with higher txid are not visible, everything else is.
- also called repeatable read by diff vendors, due to SQL standard terms.

### Preventing Lost updates
- read committed & snapshot isolation handles read only transactions in presence of transaction with writes, but what happends when 2 transactions write concurrently.
- concurrent write transaction raises several issues like dirty write , lost updates etc.
- Lost updates is when 2 **concurrent write transaction** is in *read-modify-write* cycle (updating account balance) & one of the write is clobbered, overwritten by others.
- Solutions:
- **Atomic write** : 
  - no read-modify-write (update) cycle in application code, DB take exclusive lock on the record & writes it.
  - best solution for the operations for which it works.
  - does not work well for docs or wiki updates.
- **exclusive lock** : 
  - it atomic ops is not sufficient for your operations like character update in multiplayer gaming.
  - application need to do read-modify-write using **exclusive lock.** no read/write allowed on locked records.
- **automatic detection of lost updates**
  - instead of using locks, allow transactions to run in parrallel & database detect the lost updates & abort the offending transaction & force retry for read-modify-write cycle.
  - it is handled at database level & more efficient & less error prone.
- **compare & set**
  - DBs that do not provide transaction, can provide atomic compare & set operation.
  - for single object update, compare & set, checks before update if the current value matches with read value earlier & if it does not, update is not done & needs to be retired with fresh read.
  - may not be safe for some database with snapshot isolation cause update might read old value from snapshot.
  ``` 
  -- This may or may not be safe, depending on the database implementation
  UPDATE wiki_pages SET content = 'new content'
  WHERE id = 1234 AND content = 'old content';

- **conflict resolution & replication**
  - locks & atomic operations works well with dbs where there is single copy of data, but not in replicated databases.
  - in replicated data with concurrent writes in many replicas is difficult to manage, several versions of data/writes (siblings) are maintained & appliation level or speacial merge data structures are used to resolve conflicting writes.
  - cumulative updates can be merged easily, last write wins does not work well in replicated setup.

### Write skew & phantoms
- Write Skew : You can think of write skew as a generalization of the lost update problem. Write skew can occur if two transactions read the same objects, and then update some of those object. very subtle & hard to detect anamoly. check image below. 
![write skew](/Images/write_skew.png)
- atomic writes, automatic detection does not help with write skew.
- serializable isolation level can help, if not available, exclusive locks before update is the way to go.
- phantoms : write in one transaction changes the result of select/search query in another transaction.

## Serializability
- race conditions with transactions like dirty read/write, lost updates, write skew & phantoms are some of the issues with weak isolations.
  - isolations implemented differently in diff vendors.
  - no easy way to detect race conditions.
  - no easy way to identify if particular level of isolation is safe in large codebase.
- To handle all these, **serializable isolation** is the answer.
  - strongest isolation, even though transactions are executed in parallel, the end result is as such they were executed one at time, serially. 
- Implementation:
- **Actual serial execution**
  - execute only one transaction at a time, serially on single thread.
  - cheap RAM & compute, allowed in memory DBs to load all data needed for transaction to be in memory.
  - OLTP transactions are usually short, and OLAP reads can be done from snapshot.
### Encapsulating transaction in stored procedure
- keeping user input intervined with database transaction can be disastrous, so it is removed, in some of the DBs supporting serial transaction, transaction is encapsulated in stored proc, it makes network hop smaller. Redis, voltDb uses this. partition can be used if throughput in single core is not enough, diff serial transaction is run in diff partition.
- Serial execution of transactions has become a viable way of achieving serializable isolation within certain constraints:
- Every transaction must be small and fast, because it takes only one slow transaction to stall all transaction processing.
- It is limited to use cases where the active dataset can fit in memory. Rarely accessed data could potentially be moved to disk, but if it needed to be accessed in a single-threaded transaction, the system would get very slow.x
- Write throughput must be low enough to be handled on a single CPU core, or else transactions need to be partitioned without requiring cross-partition coordination.
- Cross-partition transactions are possible, but there is a hard limit to the extent to which they can be used.

### 2-phase locking
- In 2PL, *writers don’t just block other writers; they also block readers and vice versa.* Snapshot isolation has the mantra readers never block writers, and writers never block readers, which captures this key difference between snapshot isolation and two-phase locking. On the other hand, because 2PL provides serializability, it protects against all the race conditions discussed earlier, including lost updates and write skew.
- uses shared lock for reading & exclusive lock for modify/delete/write. once transaction has taken exclusive lock, no other transaction can take shared or exclusive lock, till locking transaction completes and release the lock.
- *prone to deadlocks.*
- performace is major issue due to high amount of locking, reduced concurrency, and higher probability of deadlock.
- latency can be unpredicatle & deadlock can be frequent.
- **predicate locks** : locks result of search query.
- **index range locks** :  predicate locks do not perform well, they are approximated using index-range locking.
  - instead of locking specific records, index range locks larger portion based on ine of the index used in search query. 
    - Say your index is on room_id, and the database uses this index to find existing bookings for room 123. Now the database can simply attach a shared lock to this index entry, indicating that a transaction has searched for bookings of room 123.
    - Alternatively, if the database uses a time-based index to find existing bookings, it can attach a shared lock to a  range of values in that index, indicating that a transaction has searched for bookings that overlap with the time period of noon to 1 p.m. on January 1, 2018.

### serializable snapshot isolation (SSI)
- On the one hand, we have implementations of serializability that don’t perform well (two-phase locking) or don’t scale well (serial execution). On the other hand, we have weak isolation levels that have good performance, but are prone to various race conditions (lost updates, write skew, phantoms, etc.). Are serializable isolation and good performance fundamentally at odds with each other?
- serializable snapshot isolation is a promising algo, which can tackle this, already used in postgresql.
- Pessimistic vs optimistic concurrency control
  - 2PL & serial execution are pessimistic based on locks, where if any conflict is possible, execution is locked.
  - SSI is optimistic, it allows transaction to continue in parrallel & if any conclifct is detected while execution, offending transaction is aborted & retried.
  - like everything, it has tradeoffs, if there are many transaction is aborted due to high volume, aborted transaction just adds to the poor performance.
  - Reads are from consistent snapshots &  On top of snapshot isolation, SSI adds an algorithm for detecting serialization conflicts among writes and determining which transactions to abort.
- decision based on outdated premise
  - database reads from the snapshot based on when transaction started, & ignores ongoing transaction for snapshot isolation, and when commiting if ongoing transaction is commited & write are done, we have stale data, and need to abort. 2 cases to consider when query results might have changed.
- **detecting stale mvcc reads**
  - when database wants to commit, it checks if any of the ignored writes are now commited, if so transaction is aborted.
![stale mvcc read](images/stale_mvcc_read.png)

- **detecting write after prior reads**
  - writes happened after data was read, and commited so makes the read stale. and transaction needs to be aborted.
  - one transaction modifies another transaction's reads.
![write after prior reads](/Images/write_after_reads_txn.png)

# Troubles with distributed systems
- *Hey I just met you, The network’s laggy
  But here’s my data, So store it maybe*
- Working with distributed systems is fundamentally different from writing software on a single computer—and the main difference is that there are lots of new and exciting ways for things to go wrong.

## Faults & partial failures.
- single machines systems are *deterministic*, in absense of hardware failures they work fine, and in case of issues they crash & stops running completely.
- distributed systems can have partial failures, you can not be even sure if something succeeded or not, it makes the system indeterministic, and makes distributed systems hard to work with.
- HPC vs Cloud, 2 very different philosophies about how to handle large scale systems, meant for very diff use cases for compute intensive batch processes vs online applications based on web, where entire system should not be stopped for failure of single node.
- Accept & handle partial failure, create reliable system based on non-reliable components.

## Unreliable networks
- internet & most DC centers are async packet networks with shared-nothing systems, redundancies are used for handling partial faults.
- if you send a request & did not recieve response, it's not possible to distinguish if 
  - request was lost,
  - response was lost,
  - remote node is down
- usual way to handle this is *timeout*, wait for some time, giveuo & retry.
### network failures
- network faults are very common in distributed systems, off the shelf commodity hardware fails frequently, power failures, human errors, shark attacks, config issue, software updates, human error etc, etc.
- it is important to detect faults & handle them so system is functional.

### detecting faults
- detecting faults in distributes system is essential, i.e leader is dead in replicated system, or load balancer need to know if service is down.
- OS can help sometime with proper response if port is closed or system crashed, 
- if you have network management interface, it can help detect errors.
- but normally, it is not reliable & need to depende on timeout & retries.

### timeout & unbounded delays
- if timeout is the way to go, how long should be the timeout?
  - if timeout is big, node is down, user needs to wait long before error comm.
  - if timeout is short, and node is busy doing something else & declared dead, it just adds to already busy systems load, as dead declared nodes load needs to be transferred.

### network congestion & queuing
- if many source tries to send data to same destination, due to limited bandwidth congestion can happen.
- network congestion can cause delays, packets can be queued at switch.
- queued at operatin system level at processing node
- waiting for cpu shceduling in VCPU systems
- TCP performs it's internal flow control which also limits throughput.

 
- ``` TCP VERSUS UDP
  Some latency-sensitive applications, such as videoconferencing and Voice over IP (VoIP), use UDP rather than TCP. It’s a trade-off between reliability and variability of delays: as UDP does not perform flow control and does not retransmit lost packets, it avoids some of the reasons for variable network delays (although it is still susceptible to switch queues and scheduling delays).
  UDP is a good choice in situations where delayed data is worthless. For example, in a VoIP phone call, there probably isn’t enough time to retransmit a lost packet before its data is due to be played over the loudspeakers. In this case, there’s no point in retransmitting the packet—the application must instead fill the missing packet’s time slot with silence (causing a brief interruption in the sound) and move on in the stream. The retry happens at the human layer instead. (“Could you repeat that please? The sound just cut out for a moment.”)


- In public clouds and multi-tenant datacenters, resources are shared among many customers: the network links and switches, and even each machine’s network interface and CPUs (when running on virtual machines), are shared. Batch workloads such as MapReduce can easily saturate network links. As you have no control over or insight into other customers’ usage of the shared resources, network delays can be highly variable if someone near you (a noisy neighbor) is using a lot of resources

- Even better, rather than using configured constant timeouts, systems can continually measure response times and their variability **(jitter)**, and automatically adjust timeouts according to the observed response time distribution. This can be done with a Phi Accrual failure detector, which is used for example in Akka and Cassandra. TCP retransmission timeouts also work similarly.

### sync vs async networks
- sync : telephone lines (landline)
  - This kind of network is **synchronous**: even as data passes through several routers, it does not suffer from queueing, because the 16 bits of space for the call have already been reserved in the next hop of the network. And because there is no queueing, the maximum end-to-end latency of the network is fixed. We call this a **bounded delay.**
  - telephone lines are circuit-switched network
  - TCP/IP is packet-switched network.
- async - TCP/IP based networks are async, there is no fixed bandwidth allocation, TCP tries to use available network bandwidth & tries to send packet as fast as possible.

- networks are unreliable, congestion, queuing & unbounded delays are the norm, so picking up timout is tricky.

## Unreliable clocks
- dependent on clocks to measure multiple things & proper working of systems.
- **Durations** : timeout, response time, percentile latency, time-spent on website
- **point in time(event)** : timestamp of log/event, cache invalidation(TTL), reminder/scheduler/cron job etc,

### Monotonic vs Time of day clock
- **time of day clock** : wall clock, current time according to some calendar, seconds elapsed since *epoch*.
  - these can be in-accurate, slow/fast, & when synced with NTP(network time protocol), clocks can round backward in time. 
  - not very suitable for duration measurement. relatively coarse resolution.
- **monotonic clock** : always move forward in time, absolute value does not mean anything, fine-grained resolution, useful for measurement of duration.

### clock synchronization & accuracy
- monotonic clocks does not need to be synced, time of day clock needs to be synced.
- NTP used normally for time of day clock sync. very fickle
  - quarts clock are snesetive to temp, inaccurate & can drift(runs slow/fast), there can be drifts even when synced.
  - if clock is reset by NTP sever due to large diff, it may look like timetravel.
  - misconfigured NTPs can provide wrong time.
  - servers can be firewalled & not connected to NTP servers.
  - user can modify the time on device in case of mobiles/desktop devices. 
  - network delay/congestion in NTP sync requests.
  - leap seconds
- GPS servers & Precision time protocols are more accurate. requires significant expertise & efforts.

### Relying on synced clocks
- incorrect clocks are very easy to get unnoticed. so in systems where sync is based on clocks they need to be monitored for inaccuracies.
- **timestamp for ordering of events** 
  - in distributed, replicated systems if we depende on time of the day clock to identify which event(i.e write) occured in which order in can be disastrous, as diff nodes can have diff time in their clocks.
  - for example causally later happened write can have earlier timestamp than other write & can be overwritten by earlier write if LWW(last write wins) strategy is applied for conflict resolution.
  - even with NTP synced clocks diff in ms is quite usual & identifying most recent event can be impossible.
  - *logical clocks* like counter/vectors are used for events instead of physical clocks (wall clock, monotonic clock).
- **clock reading confidence interval**
  - wall clocks can be off by few milliseconds n reported time, even with NTP sync based on network congestion, quartz drift, temprature etc, reading can never be exact point in time.
  - it may be time with some confidence interval, like 95% sure it is 10:10.
  - Google TrueTime API in spanner gives range as time (earliest, latest), it can change in diff based on multiple things.
- **clock sync for global snapshot**
  - for snapshot isolation where short,fast write transactions & slow lon running read only transaction both needs to be supported, identifying which transaction happened before is important, can be done easily in single system.
  - but for distributed system it is difficult problem to solve, using clock for txn id is not very useful, so what can be done, area of research, spanner good setup.

### Process Pauses
- cannot rely on clock for making decisions in distributed systems, in single leader replication, if leader takes lease for some seconds as a leader & need to renew it's lease every gre seconds. it can get tricky & strange due to process pauses.
- process can be paused in different settings for multiple diff reasons as follow.
  - Garbage collector pauses in GC languages
  - context switch/scheduling in OS scheduler
  - VM can be suspended, and rebooted at some other machines
  - SIGPAUSE is sent to process 
  - costly IO operation takes long time & leader is declared dead, but does not know it.
  - too many page faults can long time & leads to thrasing in loaded system
- systems with hard real time gurantee needs can not afford any of the above, so very few lanuages & RTOS supports actual hard, mission critical real time support.
- other systems bear these pauses in diff ways.

## Knowledge, truth & lies
- ways in which distributed systems are different from programs running on a single computer: there is no shared memory, only message passing via an unreliable network with variable delays, and the systems may suffer from partial failures, unreliable clocks, and processing pauses.
- In a distributed system, we can state the assumptions we are making about the behavior (the system model) and design the actual system in such a way that it meets those assumptions. Algorithms can be proved to function correctly within a certain system model. This means that reliable behavior is achievable, even if the underlying system model provides very few guarantees.

### Truth is defined by majority
- a node cannot necessarily trust its own judgment of a situation. A distributed system cannot exclusively rely on a single node, because a node may fail at any time, potentially leaving the system stuck and unable to recover. Instead, many distributed algorithms rely on a **quorum**, that is, voting among the nodes: decisions require some minimum number of votes from several nodes in order to reduce the dependence on any one particular node.
- most commonly quorem is absolute majority.

- **the leader & the lock**:
  - many times distributed system needs only one of something like a leader, a lock/transaction, a unique id/name/resource.
  - these needs to be handled in distributed systems with care, tons of issues described earlier can make systems behave randomly with strange errors like dirty write, split brain, lost updates etc.

![distributed lock wrong](images/dist_lock_issue.png)

- **fencing token** : to avoid issues above, fencing token technique can be used.
  - Let’s assume that every time the lock server grants a lock or lease, it also returns a fencing token, which is a number that increases every time a lock is granted (e.g., incremented by the lock service). We can then require that every time a client sends a write request to the storage service, it must include its current fencing token.
  - writes with fencing token lesser than current token is discarded/ignored.
  - zoo-keeper provides facility to generate correect fencing token
  - extra work on resource/server side but it is a good idea for any service to protect itself from accidentally abusive clients.
![fencing token](images/fencing%20token.png)

### Byzantine faults
- we assume that nodes are unreliable but honest: they may be slow or never respond (due to a fault), and their state may be outdated (due to a GC pause or network delays), but we assume that if a node does respond, it is telling the “truth”: to the best of its knowledge, it is playing by the rules of the protocol. fencing tokens can help with these errors.
- if nodes are lying, it is way harder to manage, called byzantine fault, and problem of reaching consensus in this untrusting env is called byzantine general problem.
- A system is Byzantine fault-tolerant if it continues to operate correctly even if some of the nodes are malfunctioning and not obeying the protocol, or if malicious attackers are interfering with the network. This concern is relevant in certain specific circumstances. like aerospace systems, control towers etc.
- normal day to day web based systems does not need to be byzantine fault tolerant, as in we can assume all nodes in same data center are co-operating & any error is just an error not an attack, if client is trying to attack, some protection can be done at server side like authentication, authorization, query sanitization, checksum, encryption, firewall, resource limts etc.

### system model & reality
- Algorithms need to be written in a way that does not depend too heavily on the details of the hardware and software configuration on which they are run. This in turn requires that we somehow formalize the kinds of faults that we expect to happen in a system. We do this by defining a system model, which is an abstraction that describes what things an algorithm may assume.
  
- *time related models*
- **synchronous** : The synchronous model assumes bounded network delay, bounded process pauses, and bounded clock error.
- **partially sync** : Partial synchrony means that a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift
- **asynchronous** : Asynchronous model

- *node failures*
- **crash-stop** : nodes crash & stops working, gone forever.
- **crash- recovery** : nodes may crash at any moment, and perhaps start responding again after some unknown time.
- **Byzantine faults** : Nodes may do absolutely anything, including trying to trick and deceive other nodes
  
for most systems partially synchronous , crash recovery systems are most useful model.

- correctness of algo:
  - for algorithmic correcness we define it's properties, like sorting algo output where elements in left are smaller than right.
  - for fencing token algo
    - uniqueness
    - monotonic seq
    - availability
 safety & liveliness:
- Safety is often informally defined as nothing bad happens, and liveness as something good eventually happens

# Consistency & Consensus
- *Is it better to be alive and wrong or right and dead?*
- Distributed systems have 
  - unreliable networks : delayed, duplicated, dropped, out of order messages.
  - unreliable clocks: clock drifts, out of sync clocks.
  - unreliable compute : node crash, node stop(GC etc)
- one way to handle this faults is to error out & fail, if this is unacceptable, need to be **fault-tolerant:**  keeping the service functioning correctly, even if some internal component is faulty.
- The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees.
- similar to transaction in DBs :  by using a transaction, the application can pretend that there are no crashes (atomicity), that nobody else is concurrently accessing the database (isolation), and that storage devices are perfectly reliable (durability). Even though crashes, race conditions, and disk failures do occur, the transaction abstraction hides those problems so that the application doesn’t need to worry about them.
-  one of the most important abstractions for distributed systems is** consensus**: that is, getting all of the nodes to agree on something.

## Consistency gurantees
- in replicated DBs, diff replica may return diff value for same data at some point in time, most replicated systems provide **eventual consistency** : means if you stop writing to the database, and wait for some un-specified time, all reads will return same value eventually.(like convergence)
- eventual consistency is a weak gurantee & programming around it can be tricky, there can be subtle bugs in application,

## Linearizability (strong consistency)
- basic idea is to make a system appear as if there were only one copy of the data, and all operations on it are atomic.
- it is a recency property. 
- once the value is read or written, all subsequent operations will return that latest value.

![linearizability](images/linearizability.png)
- It is possible (though computationally expensive) to test whether a system’s behavior is linearizable by recording the timings of all requests and responses, and checking whether they can be arranged into a valid sequential order [11].
- it is different from serializability which make transaction appear executed on seq order.

### Relying on linearizability
- in distributed systems many functionality needs to rely on linearizability like
  - distributed locking & leader election
  - satisfying uniqueness constraint (only 1 username, filename, flight seat etc)
  - cross- channel time dependencies

### implementing linearizability
- Since linearizability essentially means “behave as though there is only a single copy of the data, and all operations on it are atomic,” the simplest answer would be to really only use a single copy of the data. However, that approach would not be able to tolerate faults: if the node holding that one copy failed, the data would be lost, or at least inaccessible until the node was brought up again.
  - single leader replication (potentially linearizable)
  - consensus algorithm (linearizable)
  - multi-leader replication (not linearizable)
  - leader less replication (potentially not linearizable)
- leaderless replication system can not provide linearizability even if using strict quorem, due to network delays. 

### cost of linearizability
- CAP(consistency, availability, paritition tolerance) theorem
- If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error (either way, they become unavailable). (CP)
- If your application does not require linearizability, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader). In this case, the application can remain available in the face of a network problem, but its behavior is not linearizable.(AP)
- The CAP theorem as formally defined is of very narrow scope: it only considers one consistency model (namely linearizability) and one kind of fault (network partitions,vi or nodes that are alive but disconnected from each other). It doesn’t say anything about network delays, dead nodes, or other trade-offs. Thus, although CAP has been historically influential, it has little practical value for designing system.
- linearizability is slow, ths is true all the time, not only in case of network partition, so it is dropped some times for performace & some time for fault tolerance.

## Ordering Gurantees
- Leader is important in single-leader replication to determine *order of writes*.
- Serializability - transactions behave as if they were executed in *sequential order*.
- timestamp & distributed clocks for consensus/leader election etc.
- deep connections between ordering, linearizability & consensus.

### Ordering & Causality
- odering helps keep causality.
- for example, question needs to be asked before answer, so there is causal dependency.
- for Action A & B, A happened before B, B happened before A, or both are concurrent. *happened before relationship is expression of causality*.
- Causality imposes order of events, if a system obeys the order imposed by causality, it is **causally consistent.**
- causal order is not a total order, some events are incomparable(concurret) & some are comparable(happened before relationship), so it is a partial order. linearizability is total order.
- Linearizability is stronger than causality, it implier causal order.

### Sequence number ordering
- maintaining causality info for arbitary operations can be impracticle, we can use timestamp or sequence number to order events, it can provide total order. 
- creating causal order based on timestamp or sequence number in leaderless or multi-leader replication is not practicle, because numbers generated on different nodes are not comparable, even if divided among nodes, no way to prove causal relation.
- **Lamport Timestamp** : pair of (counter, nodeID), every node/client keeps track of max counter it encountered, and if it receives request/response with counter > than kept counter, it is incremented accordingly. it can help maintain total order, diff than version vectors, more compact.
![lamport timestamps](images/lamport%20timestamps.png)
- The problem here is that the total order of operations only emerges after you have collected all of the operations. If another node has generated some operations, but you don’t yet know what they are, you cannot construct the final ordering of operations: the unknown operations from the other node may need to be inserted at various positions in the total order.
- total order is not sufficient for uniqueness constraint.

### Total order broadcast
- Total order broadcast is usually described as a protocol for exchanging messages between nodes. Informally, it requires that two safety properties always be satisfied:
- **Reliable delivery** : No messages are lost: if a message is delivered to one node, it is delivered to all nodes.
- **Totally ordered delivery** : Messages are delivered to every node in the same order.
- zookeeper , etcd uses total order broadcast for consensus algos, replication also needs total order broadcast, as every node apply operations in same order they will remain consistent. state machine replication. 
- linearizable compare-set & total order broadcast are equivalent to consensus.

## Distributed transactions & consensus
- Consensus is one of the most important and fundamental problems in distributed computing. On the surface, it seems simple: informally, the goal is simply to *get several nodes to agree on something*.
- Leader election
- atomic commit : transaction over several nodes/partitions

## Atomic Commit & 2 phase commit
- simpler semantics in case something goes wrong in the middle of making several writes. either commit or rollback.
- especially important for multi-object transactions & secondary indexes.
- single node transactions are already in place, once commit record in written in log, transaction is considered commited even if node crashes.
- in distributed system nodes are not able to identify if all other nodes are able to make commit. and once transaction is commited to some node, & it fails on another, it can not be reverted.

### two phase commit
- Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes—i.e., to ensure that either all nodes commit or all nodes abort.

![2 PC](images/2pc.png)

- 2PC needs a transaction manager or co-ordinator to handle **prepare & commit** phases of transaction.
- if coordinator is crashed, participants wait for it to recover & messages are written in logs to handle this recovery.
- 2PC is blocking, if coordinator fails, everyone needs to wait for it to recover.
- There is a 3PC commit proposed but not wodely used due to restrictions like bounded delay in network.
- *perfact failure detector* does not exist.
- XA transactions for heterogeneous 2 phase commit transaction
- exactly once message delivery.
- 2PC are painfully slow due to all network hopes, disk writes & locking. noSQL Dbs do not support it,
- recovering from coordinator failure
  - if logs are written recovering from coordinator failure is okay, if locks are gone & it can not decide whether commit or rollback, we need manual intervention to do it safely.
  
### Fault tolerant consensus
- The consensus problem is normally formalized as follows: one or more nodes may propose values, and the consensus algorithm decides on one of those values. must satisfy below properties. first 3 safety property, last is liveliness.
  - uniform agreement 
  - validity
  - integrity 
  - termination : needed for fault tolerance
- needs majority nodes available, assumes there are no byzantine faults.
- Raft, paxos, zab are consensus algos, they do not do formal consensus model but decide on sequence of values, so they are total order broadcast algos.
- total order reqires messages to be delivered exaclty once, in same order, to all nodes.
- epoch numbers with quorem is used for leader election & consensus.

### membership & co-ordination services
- tools like zookeeper & etcd are not used as full-fledged DBs, but as distributed key-value store or coordination & configuration services.  zookeeper is used in tons of data systems like HBase, hadoop, kafka etc.
- Zookeeper provides
  - linearizable atomic operations
  - total order for operations
  - failure detection (ephemeral nodes with timeouts)
  - change notification
- used for 
  - allocating work to nodes
  - service discovery
  - membership services

# **Derived Data**
- In reality, integrating disparate systems is one of the most important things that needs to be done in a nontrivial application.
- system of record : *source of truth*, holds authorative version of data, all events/facts are written here in normalized way. highest precedence in case of conflict/duplication.
- derived data : Data in a derived system is the result of taking some existing data from another system and transforming or processing it in some way. If you lose derived data, you can recreate it from the original source. Cache, indexes, materialized views.
  - DeNormalized, redundant & duplicate data, same data multiple point of view.
- Most databases, storage engines, and query languages are not inherently either a system of record or a derived system. A database is just a tool: how you use it is up to you. The distinction between system of record and derived data system depends not on the tool, but on how you use it in your application.

# Batch Processing
- Different type of systems
- Services (online systems)
  - A service waits for a request or instruction from a client to arrive. When one is received, the service tries to handle it as quickly as possible and sends a response back. Response time is usually the primary measure of performance of a service, and availability is often very important (if the client can’t reach the service, the user will probably get an error message).

- Batch processing systems (offline systems)
  - A batch processing system takes a large amount of input data, runs a job to process it, and produces some output data. Jobs often take a while (from a few minutes to several days), so there normally isn’t a user waiting for the job to finish. Instead, batch jobs are often scheduled to run periodically (for example, once a day). The primary performance measure of a batch job is usually throughput (the time it takes to crunch through an input dataset of a certain size). 

- Stream processing systems (near-real-time systems)
  - Stream processing is somewhere between online and offline/batch processing (so it is sometimes called near-real-time or nearline processing). Like a batch processing system, a stream processor consumes inputs and produces outputs (rather than responding to requests). However, a stream job operates on events shortly after they happen, whereas a batch job operates on a fixed set of input data. This difference allows stream processing systems to have lower latency than the equivalent batch systems.

### Batch processing using Linux Tools
- command like ```cat file | awk 'print $7' | sort  | uniq -c | sort -r -n | head -5``` for log parsing.

### The Unix Philosophy
- Make each program do one thing well. To do a new job, build afresh rather than complicate old programs by adding new “features”.
- Expect the output of every program to become the input to another, as yet unknown, program. Don’t clutter output with extraneous information. Avoid stringently columnar or binary input formats. Don’t insist on interactive input.
- Design and build software, even operating systems, to be tried early, ideally within weeks. Don’t hesitate to throw away the clumsy parts and rebuild them.
- Use tools in preference to unskilled help to lighten a programming task, even if you have to detour to build the tools and expect to throw some of them out after you’ve finished using them. 
- unix tools are easy to compose due to design decisions such as below.
- A uniform Interface : everything is a file, all commands/utils work on file as stream of bytes.
- seperation of logic & writing : the program doesn’t know or care where the input is coming from and where the output is going to.
- Transparency and experimentation.

### MapReduce & Distributed filesystem
- MapReduce is like unix tools but distributed. takes 1 or more inputs & produces 1 or more outputs. does not modify inputs & no side effects. shared- nothing architecture, used with HDFS like file systems.  
- whereas NAS & SAN needs special network hardware, HDFS can run on commodity hardware.
- Mapper : The mapper is called once for every input record, and its job is to extract the key and value from the input record. For each input, it may generate any number of key-value pairs (including none). It does not keep any state from one input record to the next, so each record is handled independently.

- Reducer :The MapReduce framework takes the key-value pairs produced by the mappers, collects all the values belonging to the same key, and calls the reducer with an iterator over that collection of values. The reducer can produce output records (such as the number of occurrences of the same URL).

- Mapper & Reducer are normally functions in programming language, HDFS nodes contain data, not the code so code is copied to the node which has data. 
![mapReduce](images/MapRedice.png)
- shuffle : partitioning sorted mapper outputs & read by reducer. reducer gives final outputs.
- workflows : in large firms, many map reduce functions are chained to perform complex workflows. there are tools to handle it better like airflow.
  
- **Reduce side Join & merge**
- **sort-merge-join** : diff datasets are mapped in 2 different mappers with same key, and values are different, those are sorted in pushed to reducer, which leads to both set of data with same key adjacent, called secondary sort.

- **Mapside Join**
- Broadcast hash join
  - small dataset is merged with the large one, where small one can fit in memory and broadcasted to all the nodes.
- partitioned hash join
  - both datasets to be joined are partitioned in similar way, so same subsets go to paritioned nodes & can be joined

### output of batch processing
- batch processing output is neither OLTP(small number of records to show to end users) or OLAP(analytics report/metrics for manager, business decision, EOD reports), so what it is used for?
- Search index
  - Google used it to build search index, Lucene still uses it.
- key value stores
  - MapReduce outputs also used fot machine learning algos like recommendation systems or classifiers. those reads from database usually.
  - MapReduce jobs are all or nothing type, so writing to DB from batch job is bad idea, usually database files are created in batch job and dumped to HDFS, databases supporting database files like voldemort start reading from these files.

- ### distributed database & Hadoop
  - Massively Parrallel Processing have been a thing long before hadoop, Hadoop/MapReduce is like Distributed Unix tools, with default sort for every reduce & map job.
  - Databases require you to structure data according to a particular model (e.g., relational or documents), whereas files in a distributed filesystem are just byte sequences, which can be written using any data model and encoding. They might be collections of database records, but they can equally well be text, images, videos, sensor readings, sparse matrices, feature vectors, genome sequences, or any other kind of data.
  - Sushi Principal - raw data is better, consumer decides how to use it, schema on read.

## Beyond MapReduce
- MapReduce simple to understand, but cumbersome to implement complex workflows, abstractions over it like Pig, Hive helps with some tasks like Join. but still fundamentally slow.
- there are other tools which may work significantly faster for specific use cases.

### materialization of intermediate states
- in MapReduce jobs are independent, if one job/task needs output from other as input, it can start only after first is complete. if multiple jobs consume the output of same job, keeping at well known location is good idea.
- but sometimes, only one job need output of a job, but mapReduce still need to write out it to HDFS, it is called materialization of intermediate state.
- compared to unix pipes
  - chained job can only start after first one is completed, some part of distributed job can fail, runs really slow, hampers the entire process flow.
  - HDFS files are replicated, so written intermediate state is also replicated, costs time, memory.
  - some mapper are redundant, and can be easily be part of reducers.

### dataflow engines
- spart, tez, flink
- they handle entire workflow as one job instead of breaking it up into independent jobs.
- no strict rule of alternating map & reduce jobs, more flexible way to process data, called operators.
- it partition input and process in single thread, and copy output from one function to another.
- supports partition with sort, sort-merge-join etc.
- not every stage needs to write data to HDFS, can be shared via memory.
- not every stage/operator is seperate JVM process, save with bootup time.
- executor know entire flow, so can be optimmized.
- no need to wait for job/operator to be done, next stage can start as soon as data is available.
  
- Fault Tolerance : mapReduce always dump intermediate state, so fault tolerance is very straight forward, rerun the job.
  - spart, tez, flink do not dump all intermediate state, they use some structures to store metadata about computation.
  - these engines check the last available state & rerun the job from the portion needed, but these needs to be deterministic, if data is random or out of order, downstream operator can face ambiguity.
  - sometimes if intermediate state is much smaller or operator is very compute intensive, writing intermediate state is better option.

### Graphs & iterative processing
- It is also interesting to look at graphs in a batch processing context, where the goal is to perform some kind of offline processing or analysis on an entire graph. pagerank as an example.
- graphs processing do not work well in mapReduce model. As an optimization for batch processing graphs, the bulk synchronous parallel (BSP) model.
- pregel model :
  - In each iteration, a function is called for each vertex, passing the function all the messages that were sent to that vertex—much like a call to the reducer. The difference from MapReduce is that in the Pregel model, a vertex remembers its state in memory from one iteration to the next, so the function only needs to process new incoming messages. If no messages are being sent in some part of the graph, no work needs to be done.
  - messages are send over vertex over edges/network, in every iteration. supports exactly one delivery
- if your graph can fit in memory on a single computer, it’s quite likely that a single-machine (maybe even single-threaded) algorithm will outperform a distributed batch process

### high level APIs & languages

# Stream Processing
- *A complex system that works is invariably found to have evolved from a simple system that works. The inverse proposition also appears to be true: A complex system designed from scratch never works and cannot be made to work.*
- Batch process works on assumption that data is bounded - known & finite. batch processes works on fixed time interval. so data changes between specified time is not reflected.
- to reduce delay, we can process data more frequently let's say per second, or process as the event as it happends, this idea is stream processing, handling/processing stream of events as it comes.

## Transmitting event streams
- In batch process input & output is usually a file.
- When the input is a file (a sequence of bytes), the first processing step is usually to parse it into a sequence of records. In a stream processing context, a record is more commonly known as an **event**, but it is essentially the same thing: a small, self-contained, immutable object containing the details of something that happened at some point in time. An event usually contains a timestamp indicating when it happened according to a time-of-day clock.

### messaging systems
- A common approach for notifying consumers about new events is to use a messaging system: a producer sends a message containing the event, which is then pushed to consumers. 
- Within this publish/subscribe model, different systems take a wide range of approaches, 2 imp questions to answer
  - What happens if the producers send messages faster than the consumers can process them?
  - What happens if nodes crash or temporarily go offline—are any messages lost?
- Direct messaging between producers & consumers
  - messaging systems use direct network communication between producers and consumers without going via intermediary node. like UDP multicast, zeroMQ etc.
  - does not have very good fault tolerance, what if consumers are down. requires application code to handle message loss.
  
- Message Brokers
  - Brokers act as middleware, producer sends messages to broker, consumer read from broker.
  - broker can save/persist messages based on configuration
  - pub/sub model, many-to-many relation.
  - consumers are async in this setup.
- Multiple consumers
  - Load Balancing - each message is given to seperate consumer, to parrallize work
  - Fan-out - each message given to all consumers.
  ![multiple consumers](images/multi_consumer.png)
- Acks & redelivery
  - brokers waits for ack from consumer before removing message from queue, if it does not recieve ack it can send messgae to another consumer.
  - it can lead to ordering issue in case redelivery with load balancer.

### partitioned logs
- message brokers are transient, once message is processed it is deleted, DB & filesystem are opposite in nature, once something is persisted, it stays until explicitly deleted.
- for batch process, repeating the task with same input is non-destructive & readonly & easily repeated.
- log based message brokers can combine these 2.
- a producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially.
- to achieve scalability logs can be partitioned, which can behave independently, topic is group of such partitions. within each partition broker assign monotonically increasing sequence number.
- Apache Kafka, Kinesis are examples.
  ![log based message brokers](images/log_based_message_broker.png)
- it works as append only los, messages are added to logs and written to disk, amount of messages buffered are based on disk size and can save messages for days for really large disk.
- it is like ring buffer or circular buffer.
- if some of the consumer fall behind head of the buffer, it can lost some messages but do not affect others.
- replaying old message is really easy, just reset the offset and rerun.qwfr2

## Database & Streams
- event is a record of something that happened at some point in time. The thing that happened may be a user action (e.g., typing a search query), or a sensor reading, but it may also be a write to a database. there is a fundamental connection between databases & event streams.

### Keeping systems in sync
- there is no single system that can satisfy all data storage, querying, and processing needs. In practice, most nontrivial applications need to combine several different technologies in order to satisfy their requirements: for example, using an OLTP database to serve user requests, a cache to speed up common requests, a full-text index to handle search queries, and a data warehouse for analytics. Each of these has its own copy of the data, stored in its own representation that is optimized for its own purposes. all those systems needs to kept in sync.
- **dual writes** in which the application code explicitly writes to each of the systems when data changes: for example, first writing to the database, then updating the search index, then invalidating the cache entries (or even performing those writes concurrently).
- dual writes are prone to race conditions. even in single leader replication, different systems have their own leaders. using atomic writes / 2 phase commit is very expensive.

### change data capture
- replication log of database has been implementation detail, there has been no documented way to parse writes from replication logs.
- change data capture (CDC), which is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems.
- ![CDC](images/change_data_capture.png)
- log consumers are derived data systems. Essentially, change data capture makes one database the leader (the one from which the changes are captured), and turns the others into followers. A log-based message broker is well suited for transporting the change events from the source database to the derived systems, since it preserves the ordering of messages.

- log compaction : log based message brokers & CDC can use compaction strategies similar to log structured storage engine.
  - it allows adding new derived system without need to taking snapshot of database. apache kafka provides this.
- API support for CDC - kafka connect & some other DBs.

### Event sourcing
- similar to CDC, but different abstractions as immutable record of events.
- In event sourcing, the application logic is explicitly built on the basis of immutable events that are written to an event log. In this case, the event store is append-only, and updates or deletes are discouraged or prohibited. Events are designed to reflect things that happened at the application level, rather than low-level state changes.
- event log in itself is not useful, application need to have read logic to acertain current state of the system from the events, diff from CDC.
- applications that use event sourcing need to take the log of events (representing the data written to the system) and transform it into application state that is suitable for showing to a user (the way in which data is read from the system).
- log compaction does not work with event sourcing systems. 
- command vs event : if user perform any action it is command, which is than validated, if successful it becomes event,(fact). all consumer needs to accept the event.
  
### state, streams & immutability
- batch processing benefits from the immutability of its input files, so you can run experimental processing jobs on existing input files without fear of damaging them. This principle of immutability is also what makes event sourcing and change data capture so powerful.
- The key idea is that mutable state and an append-only log of immutable events do not contradict each other: they are two sides of the same coin. The log of all changes, the changelog, represents the evolution of state over time.
![stream](images/data-event.png)
- *Transaction logs record all the changes made to the database. High-speed appends are the only way to change the log. From this perspective, the contents of the database hold a caching of the latest record values in the logs. The truth is the log. The database is a cache of a subset of the log. That cached subset happens to be the latest value of each record and index value from the log.*
- **Deriving several views from the same event log**
  - by separating mutable state from the immutable event log, you can derive several different read-oriented representations from the same log of events.
  -  you gain a lot of flexibility by separating the form in which data is written from the form it is read, and by allowing several different read views known as CQRS(command query responsibiity segregation)
- deleting data for regulatory reasons gets tricky for eventlog systems, 
- this systems are async in nature, so read your own write needs kind of distributed transactions, on other hand multi-object writes can be simplified by modelling events as per user action.

### Processing streams
- stream comes from events & transported via messages in, there are 3 options to process it.
  - You can take the data in the events and write it to a database, cache, search index, or similar storage system, from where it can then be queried by other clients.
  - You can push the events to users in some way, for example by sending email alerts or push notifications, or by streaming the events to a real-time dashboard where they are visualized. In this case, a human is the ultimate consumer of the stream.
  - You can process one or more input streams to produce one or more output streams. derived streams.  a stream processor consumes input streams in a read-only fashion and writes its output to a different location in an append-only fashion.
- one cruical difference from batch job is, stream never ends, sorting does not make sense, fault tolerance needs to be different than just a rerun/restart.

- Uses of stream processing
  - traditionally used in monitoring like trade capture, fraud detection, anamoly detection etc.
- Complext event processing
  - search for specific event patterns in event streams, roles are reversed from database where queries are transient and data is stored, here CEP is stored long term & data is a stream. Samza, SQL Stream.
- Stream analytics
  - aggragation & statistics like 99 % percentile, avg. mean over window.
  - probablistic algos like bloom filter, hyperLogLog are used for performace optimizations.
  - Apache flink, storm, kafka streams, samza 
- Materialized views
  - maintaining materialized views : deriving an alternative view onto some dataset so that you can query it efficiently, and updating that view whenever the underlying data changes..
- Search queries on stream 
  - like elastic search but on on-coming events, used for alert systems, when specific event happened, like simlified CEP.

### Reasoning about time
- for batch process, times to be looked at are embedded in event, system clock does not matter much, event streams usually work on time windows, many frameworks uses local system clock to determine window, works fine when difference between event creation & processing is small, breaks down in case of significant lag.
- Event time vs process time : there might be delay in processing an event, it is required to not confuse between two, which may lead to false alarms, spikes in case of delay due to any reason like restart, contention, crash, network issue.
- knowing when window ends, straggeler events: delayes events which comes after window is over, either discard or update the output.
- 3 event times
  - event occured on local device
  - event sent to server from local device
  - event recieved on server
  - subtract 2nd from 3rd, to identify difference/drift and adjust event time in 1st accordingly.
- **Type of Windows**
  - tumbling windows - like 1 minute window like 12:01 to 12:02
  - hopping windows - fixed sized windows with overlap.
  - sliding window - sliding window of fixed length, events are discarded as window slides.
  - session window - 

### Stream joins
-  the fact that new events can appear anytime on a stream makes joins on streams more challenging than in batch jobs.
-  Stream-Stream Joins
   -  like joining search & click results for a URL, needed in ads. streams needs to maintain some state for this joins.
-  Stream table joins (enriching)
   -  like user activity is 1 stream & needs to be enriched with user profile data, can be done via querying DB, but time consuming, keep user data as another stream and do join, any change in user data is also an event now.
-  Table - table join : keep materialized view maintained
  
### Fault tolerance
- Batch process achieve good fault tolerance by treating input file as immutable & writing output files to HDFS, any failed task can be retried without much issue. for stream processing waiting until a task is finished before making its output visible is not an option, because a stream is infinite and so you can never finish processing it.
- Microbatching : process streams in small time-windows like 1 seconds, which leads to small batches to write outputs & does not delay processing much.
- checkpointing : writes output periodically to disk, in case of crash restart processing from checkpoint.
- if there are side effects, none of the options helps to achieve exactly once semantics.
- Atomic commits : does not work well with distributed heterogeenous systems, bit can work with tight coupled systems like kafka.
- **Idempotence** : Our goal is to discard the partial output of any failed tasks so that they can be safely retried without taking effect twice. An idempotent operation is one that you can perform multiple times, and it has the same effect as if you performed it only once.
  
# Future of data processing
## Data integration
- most applications needs to consume same dats in different context & way, most systems are good at specific use, even 'general-purpose' systems like SQL DBs are not good at all usage patterns. applications needs to combine diff tool good at diff use for efficient systems.

### Combining specialized tools by deriving data
- need for data integration became apparent only when you zoom out & consider data flows for entire org.
- be clear about input/output/source/flows of data, what is source of truth, which is record system (DB) vs derived data systems, needs to be applied in same order to make it consistent.
- updating derived systems based on event log can be made deterministic & idempotent.
- total order based on event log works for the cases where ordering can he handled by single leader,  it is still an open research problem to design a consensus algo that can scale beyond the throughput of a single node & work well with geo-distributed systems.

### Batch & stream processing
- goal of data integration is to make sure data ends up in the right form in right places. it  requires consuming inputs, transforming, joining, filtering, aggregating, training models, evaluating & eventually writing the outpus. batch & stream processors are the tools to achieve this.
- batch & stream processing has functional flavor, output only depends only read only inputs & processing is pure deterministic functions with no side effects, apart from append only outputs, helps with fault tolerance.
- lambda architecture
  - design where systems do both stream & batch processing on same events.

## unbindling databases
- OS, Database & Hadoop all store data & allow to query the same data.
- OS & DB have very diff philosohies, OS is very thin wrapper on hardware resource & DB is simple in the way it is short declarative query ti access the database.

### Composing data storage tech
- DBs provide secondary indecxes, materialized view, full text search index, replication logs.
- stream & batch processing derived data systems are similar to above features.
- there are 2 avenues by which diff storage & processing tools can be composed in cohesive system
- federated database : unifying write
  - write data to multiple systems & provide unifying read interface on top of all that, again creating index over all source & replication among them can be tricky
- unbundle database : unifying writes
  - changes flow through events & CDCs, like unbundling DBs index building feature. like unix philosophies, do one thing well & provide unified interface for processing IO.
- unbindling using event logs with idempotent writes help with fault tolerance as async nature allows systems to work independently, and diff teams to work independently.
- but do not optimize prematurely, if single systems needs all your needs, do it. as making application with many moving systems has infra cost, learning curve & one more failure point.

### Designing application around dataflow.
- application code as derivative function
- when 1 dataset is derived from another it goes through transformation like seconday index, search index, machine learning systesm, cache etc.
- When the function that creates a derived dataset is not a standard cookie-cutter function like creating a secondary index, custom code is required to handle the application-specific aspects. And this custom code is where many databases struggle. 
- seperation of applicaition code & state
- data flow : interplay between state change & application code
  - ordering of message delivery & fault toerance are important
- stream processors & services
  - dataflow can look like similar to micor-service approach, but instead of sync asking for changes from databases, all changes needed to are subscribed & changes automatically.
  - Subscribing to a stream of changes, rather than querying the current state when needed, brings us closer to a spreadsheet-like model of computation: when some piece of data changes, any derived data that depends on it can swiftly be updated.

### Observing derived state
- write path : batch/stream processors creating derived data
- read path : reading derived data & post processing for query/user/reports.
![derived data](images/observ_derived_data.png)
- Materialized views & caching
  - search index : creating index is write path, searching based on index and bool operator is read path.
  - we can see that an index is not the only possible boundary between the write path and the read path. Caching of common search results is possible, and grep-like scanning without the index is also possible on a small number of documents. Viewed like this, the role of caches, indexes, and materialized views is simple: they shift the boundary between the read path and the write path. They allow us to do more work on the write path, by precomputing results, in order to save effort on the read path. 
- we can think of the on device state as cache of state on the server.

## Aiming for correctness
- for stateless services with readonly data, faults, failures is not a big deal, just rerun. for databases which stores data indefinately, rerunning is not an option and correctness is paramount.
- for SQL DB, transactions properties like atomicity, isolation, durability provide helps building correct applications.
- consistency is often talked about, but poorly defined. it is very difficult to determine if it is safe to run application at particular isolation level & replication configurations.
- distributed systems with need for scale is hard to handle, if you can tolerate few inconsistencies, can be managed. if needed strong gurantees serializability & atomic commits are costly affair.

### end to end argument for database
- serializable transactions do not save from faulty application code/bugs. immutability is useful , but not silver bullet that solves everything.
- exacltly once execuation of operations
  - we do not want to charge customer twice or update some matric multiple times even in case of some fault. making operation idempotent is effective.
- duplicate suppression
  - TCP handles duplicate/lost messages/packets using sequence numbers, but database transactions can not work like that, what if client lost the connection before the transaction was commited, does he retry the transaction, was it commited or aborted. what if user resubmit the form to server, where for server it is fresh request but for database same data is begin updated again.
- Uniqly identifying request
  - to make request idempotent accorss network hops, transactions only is not enough, need to consider **end to end flow**.
  - UUID helpes with this task, you can store UUID for request in event log/DB table & reject the request if same UUID is encountered again.
- end to end argument.
  - communication channel can not provide end to end feature.
  - like TCP can not help with duplication due to user submitting twice.
  - encryption & checksum only helps between 2 endpoints, not entire flow of packet.

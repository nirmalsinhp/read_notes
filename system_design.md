## NetFlix Zull
- how to send notification
  - Polling : client asks for data, REST GET
  - Push messaging : server push messages to client,
    - WebSockets 
    - SSE : server set events
  

## 
- elastisearch
  - inverted index,
  - text based search
  - geospatial, quad tree , 
  - node query caching
- redis
- server sent events
  - HTTP REST - client to server
  - SSE - server to client persistent
- choke point
  - virtual waiting queue

## Geo spatial data
  - Postgres GIS
  - Quad tree
  - Redis with Geohash - heavy write

## temp blocking like seat
  - cron job to change status - bad idea
  - distributed lock in Redis with TTL

## DBs
- SQL - use for normal entity relationship
- NOSQL
  - document - where entire entity are added & retrived like tweet
  - S3 - all static/media data
  - wide column/ Cassandra - write heavy like messaging
  - search index - text search like elastisearcj
  - redis - distributed lock with TTL, cache, queue
  - Graph - for Graph relations

### ELK 
- elastic search - store logs
- log stash - analyze/read
- graphana - visual presentaion

Rate Limiter:


### Live Streaming
- Sources
  - Open Broadcast protocol
  - browser
  - mobile
- RTMP - Real time messaging protocol - Adobe - TCP
- SRT - Secure Reliable transport  - UDP
- POP servers 
- Adaptive Bit Rate streaming - Transcoding
- Packaging format
- HLS 
- DASH

### Misc
- optimistic locking strategy
- Scaling
  - vertical
  - horizontal
  - Work Distribution
  - Data distribution
- consistency
  - string
  - weak
  - fine tune as needed, not a property of entire system
- Locking
  - Avoid if can
  - Distributed lock
-  Indexing is the process of creating a data structure that makes reads faster.
- There are many different types of indexes but the principle is the same: do a minimal amount of up-front work so that your reads can be extra fast.
  - DB index - DB reads
  - GeoSpatial index - location data
  - Full text index search - text search
  - vector index - imaging
- ElasticSearch is our recommended solution for these secondary indexes, when it can work. ElasticSearch supports full-text indexes (search by text) via Lucene, geospatial indexes, and even vector indexes. You can set up ElasticSearch to index most databases via Change Data Capture (CDC) where the ES cluster is listening to changes coming from the database and updating its indexes accordingly. 
- Communication protocol
  - Internal 
    - Grpc
    - HTTPs
    - Queues
  - External
    - consider, protocol, who initiate connection, latency, amount of data
    - HTTP(REST) : request response
    - WebSockets : 2 way communication, difficult to maintain
    - Long Polling : real time updates, keep connection open, server sends data
    - SSE : server keeps sending data to client, 1 way server to client, more effieicient
- Security
  - Authentication & Authorization : API GateWay, Oauth, SSO, JWT, SAML, RBAC
  - Encryption : in transit, SSL/TSL & Rest : encrypted
  - Data Protection
- Monitoring
  - Infra Monitoring : Data Dog, monitor CPU, NetWork, Memory
  - Service : Latency, throughput, Errors at service level
  - Application level : Active users, sessions, analytics

## Core Tech
### DB
- RDBMS : ACID, Indexes, Joins 
- NOSQL : Graph, Key-Value, Document, Column
  - Horizontally Scaled, Consistent hash or shards
  - mostly schemaless, schema at read opposed to schema t write in SQL
- Blob :
  - Cheap, blob storage like S3, infinitely scalable, versoning & Access control, no mutability, multipart for large files
- Search Optimization
  - SQL query with %like% can be slow for hugh dbs.
  - Tools like elastisearch uses Inverted index
    - word1 : {doc1,doc23}
    - Tokenization, stemming, fuzzy search using regEx, Edit distance

### API Gateway
- like a face to your micro-service arch
- Cross Cutting concerns like Authentication, routing, rate limiting, load balancing, protocol switch

### Load Balancer

### Queue
- Aysnc Communication, task based loads, batching, 
  - Order
  - Retry
  - Dead Letter Queues

### Streams/Event Sourcing
- Good for real time ingestion & Analytics

### Distributed locks
- locks a resource temp, may be using redis / zookeepre, useful for tickting, uber etc 
### Distributed cache
- Store expensive operation result in memory
  - like Aggregation result
  - expensive DB query
  - user session
- Cache eviction policy
  - LRU
  - FIFO
  - LFU
- write strategy
  - write through , db && CACHE
  - write around, DB, DB ->CACHE
  - write back, cache, cache->DB

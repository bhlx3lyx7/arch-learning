# Google SWE teaches systems design

[Youtube link](https://www.youtube.com/playlist?list=PLjTveVh7FakKjb4UYzUazqBNNF-WGurXp)

## EP1: database design
- durability -> hard drive -> cheaper but slow than SSD -> sequential operations
- implementation
	+ list
	+ append-only log
	+ hashmap: random write/read, not good for disk
	+ indexes: fast read, slow write
- indexes
	+ hash index
		* all the keys in mem
		* bad for range queries
	+ LSM tree (log-structured merge-tree) and SSTable (Sorted Strings Table)
		* LSM tree in mem, SST file in disk, with necessary compaction
		* bloom filter for faster existence check
		* high write throughput; good for range queries due to internal sorting in index
		* slow read for old or non-exist key; background merge process
	+ B-tree
		* fast read; good for range queries as data is sorted in index
		* slow write, index in disk

## EP2: single leader replication
- leader write, all nodes read
- increase avaiability
	+ add a follower
	+ on follower crash
	+ on leader crash
- types of replication
	+ sync, strong consistency
	+ async, eventual consistency
- eventual consistency
	+ read your own write
		* read from leader or up to date replica after a client write
	+ monotonic read
		* hash user id and map to a given replica
	+ consistent prefix read
		* causal events in the same partition
		* sequential information to order the events
- replication log implementation
	+ copy over sql statement
		* sql statement might be non-deterministic
	+ use internal write ahead log
		* not scalable if db engine changes, like serialization bytes changed
	+ use logical log
		* describe which rows were modified and how, allows for db engien change in future
- conclusion of single leader replication
	+ pros: single leader write ensures consistency across replicas
	+ cons: write throughput is low; need to partition and have a leader per partition

## EP3: multiple replication
- multiple leader topologies
	+ circular: easy to fail if any node crash
	+ star: easy to fail if central node crash
	+ all-to-all: write out of order due to race condition
- multi leader replication tradeoffs
	+ pros: can have a leader in each DC, makes a global service more available; write throughput not limited by a single node
	+ cons: deal with write conflicts between multiple leaders
- conflict resolution
	+ conflict avoidance
		* all writes to the same item go to a given leader
		* not always possible if leader is down or configuration changes
	+ last write wins
		* record with latest timestamp wins
		* pros: easy to implement
		* cons: timestamp from client or server; writes could be lost; clock skew on servers means they are not synced
	+ on read
		* when db detects conflict writes, store both
		* return both values to user on the next read, to manually merge values and store the merged result back
	+ on write
		* when db detects conflict writes, call snippet of code to merge them, often specified by application
		* this is the idea between CRDT (conflict free replicated data types)
- detect concurrent writes
	+ concurent writes means no awareness of each other's write
	+ awareness of other write indicates a causality relationship
	+ detect concurrent writes is about keeping track of what a client has seen from the db before making a write
- version vector
	+ keep track of an increasing version number for each key for each replica
	+ e.g. `[1, 0]` and `[0, 2]` means two version vectors seen and updated by two users
	+ when reads, the version vector of a key is given
	+ when writes, the most recently read version vector of the key should be also passed, and the db supplies a new one
	+ two values have a happens-before relationship if one version vector is strictly greater than the other, or they are concurrent
	+ concurrent values can either be merged or kept as siblings in db
	+ merging version vectors means taking the max of both version vectors at every index

## EP4: leaderless replication
- any replica can accept a write from a client
- send reads and writes to all nodes in parallel, responds when predefined threshold of nodes return success
- keep data up to date
	+ anti-entropy: background process looks at multi nodes and stored values, use version numbers to make sure each replica holds the most up to date copy of the data
	+ read repair: read from multi nodes in parallel, take the most up to date piece and propagate it to the other replicas which had outdated data
- quorums: w + r > n; implicitly, quorum write doesn't have guarantee of order
- quorums are not perfect
	+ if less than w writes succeed, the client is told a write failure, but the succeeded nodes will not undo the write
	+ if a node with new value crashes, and we restore it by syncing from a node with old value, the key will lose the up to date value
	+ there can still be write conflicts, similar to the multiple replication, need to resolve conlicts
	+ sloppy quorums: failed nodes come back will not transfer the updated data back to the nodes
- conclusion
	+ pros: good write performance with small write quorum number
	+ cons: slower read due to multi queries; still need to handle write conflicts; quorums are not perfect, which is not strong consistency

## EP5: database sharding/partitioning
- share data, share traffic; to avoid hot spots
- methodologies
	+ by ranges of keys: not necessarily even, sorted within a partition to better support range queries
	+ by ranges of hash of keys: keys are evenly distributed, range query has to check each partition
- secondary index in partitions
	+ local index: a secondary index only holds data in local partition
		* fast on write
		* slow on read because query through the index has to accumulate all partitions
		* however, index is mainly for better read performance, so it is not so good
	+ global index: the secondary index is partitioned across nodes, like data set
		* fast on read, a secondary index value can be found in one partition
		* slow on write, a write needs to update various secondary indexes across multi partitions
		* may require distributed transaction for consistency
- rebalance
	+ fixed number of partitions: the number should be reasonable
	+ dynamic partitioning: auto split; might fall into unnecessary rebalancing due to just a slow network
	+ fixed number of partitions per node: each node has a certain partition number; similar to consistent hashing to avoid unfair data splits
- generally requires coordination service or gossip protocal to keep track of partitioning meta information

## EP6: ACID Transactions
- isolation is the main reason of transaction penalty performance
- serializable isolation
	+ actual serial execution: single thread, with all the following conditions:
		* in mem db
		* use a stored procedure
		* keep all transactions limited to one partition
	+ two phase locking (2PL): shared lock or exclusive lock for read or write
		* predicate lock: lock all rows matching a given search condition, even the ones not yet exist; performs poorly
		* index range lock: alternative of predicate lock, to lock on index, to describe the search condition
		* dead lock of 2PL: db must detect dead lock and abort one of them, to finish the other first, then retry the aborted one
	+ serializable snapshot isolation: concurrent transactions run as there is no lock, only revert a transaction if a concurrency bug is detected
		* all reads occur from a snapshot of db
		* write A uncommitted -> txn T read A -> write A committed -> txn T write (should abort, because txn T read A out of date)
		* keep track of all txns read an item, if another txn writes it, abort all txns read it
		* it works better if not many concurrency issues; but could be many retries if lots of concurrency bugs, then 2PL may be better

## EP7: weak forms of isolation
types of concurrency issues:
- dirty read: read uncommitted value; read committed to resolve it
- dirty write: write to uncommitted value; should write to committed value, exclusive lock for write
	+ read committed isolation
- read skew (non-repeatable read): multi read inconsistent state during a txn
	+ snapshot isolation: each txn id is saved with the rows it writes, value can be seen if it has the highest txn id while smaller than reader txn id
- lost update: concurrent txns to read value, change and update, there might be update lost (for single row)
	+ atomic write operation: CAS, exclusive lock
	+ explicit lock: hard to reason about, lead to many bugs
	+ automatic db detection: use snapshot isolation to detect lost updates (can detect the value it is about to write has been changed by other txn id), and then rollback and retry
	+ these solutions doesn't work in multi-leader/leaderless replication, because they assume one copy of data, better to store conflicts as siblings or use custom resolution logic
- write skew: concurrent txns to read a set of rows, make decision and update, which might impact the read set of rows (for multi rows)
	+ predicate lock: lock to all the rows reading to form the predicate, but the non-exist row can not be locked
- phantoms: same issue as write skew, but invariant are broken when both txns create a new row, nothing can be locked
	+ materialize conflicts: create a physical row or object, then concurrent txns can lock on it

types of isolations/solutions:
- read committed: prevent dirty read/write
- snapshot isolation: prevent read skew, and be adapted to detect lost udpate
- predicate locks and materialize conflicts: prevent write skew and phantoms

## EP8: SQL and its pitfalls / pitfalls of relational databases
- scale out poorly when sharded
	+ on writes to many shards may need distributed transactions
	+ on reads to many shards involves many network calls
- transaction abstraction and locking is slow
- b-tree is slow for write (since it go to disk)
- schema needs to be predefined
- generally single leader replication
- NewSQL product like VoltDB, Google Spanner, TiDB tried to improve the scalability of the relational model

## EP9: data warehousing / analytic database
- decouple analytics db from transactional db, for analytic queries
- column oriented storage fits for analytic db
- materialized view caches pre-computed analytic data set; data cube is similar, it pre-computes on multi dimensions

## EP10: unreliable clocks
- time drift between nodes leads to time difference
- machine time is unreliable for event ordering

## EP11: fencing tokens
- process pause: a process is running and be preempted by another "stop the world" process for many seconds
	+ long GC
	+ virtual machine moved from one host to another
	+ a user closes his laptop lid
- during process pause, the lease of a distributed lock or leader election could be expired, another process can get the lock or leader role
- fencing token: assign a monotonically increasing id for the lock or leader election, write data to storage with the id, if storage receives a lower fencing token value than current one, reject it
- fencing token is an important machenism for leader election (leader epoch), to make the majority decision over a single node decision

## EP12: linearizability and ordering
- linearizability = strong consistency
	+ always up to date
	+ huge performance hit
- linearizability: act as if there is only one copy of the data, there should be a total order of operations on the data; the state should be able to be expressed by all the operations in some set order, with causality preserved, and concurrent operations can be in any order, but should be in the same order in each replica
- causality can be expressed by just a partial order, so linearizability > causality
- how to create a total order
	+ single leader replication: WAL (write ahead log)
	+ leaderless / multileader replication: lamport timestamp; version vector
- lamport timestamp vs. version vector
	+ lamport timestamp take less space (O(1) < O(k))
	+ version vector can express concurrent operations since it is partial ordering, whereas lamport timestamp is total ordering
		* lamport timestamp loses information as it simply keeps the largest timestamp
		* lamport timestamp has no ability for custom merging/sibling logic
- pitfalls of lamport timestamp
	+ it gives a total order retroactively (not in realtime, just after synced from all replicas), so at the moment of client requests, they could both receive success response, but after convergence, one of them would be a failure
- total order broadcast: the protocal to propagate the total order in time, without data loss or misorder
- consensus: the equivalant to solving the total order broadcast or linearizable storage

## EP13: two phase commit
- 2PC (two phase commit): involve coordinator, participant nodes; for distributed atomic transaction
	+ phase 1, prepare
	+ phase 2, commit/abort
	+ participant nodes should guarantee the phase 1 response can be committed in phase 2 if it responses ok
	+ coordinator should keep retrying to guarantee the phase 2 executed and succeed
- 2PC lacks fault tolerance
	+ coordinator is single node or single leader replication, if it is down, the whole system can not process
	+ if participant node is down, the transaction can not be committed or aborted until the coordinator can reach it
	+ so 2PC always requires the availability and consistency of coordinator and participant nodes
- heterogenous vs. db internal transactions
	+ heterogenous: among different types of db, using an API called XA, hard to optimize for performance
	+ db internal transactions: on the same type of db, more information can be shared, easier to optimize performance

## EP14: Raft
- one leader sends all writes to follower nodes
- once majority (quorum) responses are received, the leader tells the followers to commit
- if a leader presumed to be dead, a follower node begins an elelction to be a new leader with a higher term number (fencing token)
- leader election
	+ leader sends heartbeat periodically to followers
	+ if heartbeat timeout, a follower becomes a candidate to start new leader election, votes for itself
		* heartbeat timeout is randomized over a range, in case all the followers vote for themselves at the same time
	+ the other nodes will vote for a candidate if the term number is higher than local, or the candidate log is more up to date, and the node has not voted for any other candidate in this term; else it rejects the vote
	+ a candidate becomes a leader if it receives majority votes
- broadcast messages
	+ leader receives a message, it writes to local log, not committed yet
	+ send it to all the followers, the sending function can be called as replicateLog
	+ replicateLog function is periodically called, act as heartbeat, to keep logs in sync, and ask other nodes to commit messages that should be committed
	+ leader keeps track of sent messages number for each other node, and split its local log into a prefix and suffix, for each node as well
		* prefix: the logs has been sent to the remote node
		* suffix: the logs not yet sent to the remote node
		* the prefix of some nodes could be wrong, then leader can adjust it when receiving follower response
	+ when a follower receives the suffix logs, together with the term number of the last prefix log, it checks the term number matches local log or not
		* if matches, just go and copy the suffix logs, response ok
		* if not match, response reject
	+ when the leader receives response from a follower
		* if success, it keeps track and waits for quorum positive responses to commit the message and notify followers
		* if fail, it must try the write again with a smaller prefix, to overwrite the inconsistent logs on the follower node
		* also possible, the follower's response may tell that it has a higher term number write, then the leader steps down
- comparing with 2PC, Raft is good for making replicated logs, but not so great for cross partition atomic transactions

## EP15: batch processing
- bounded data set
- HDFS, MapReduce
	+ sort merge join
	+ broadcast hash join
	+ partitioned hash join
- pitfalls of MapReduce
	+ data skew
	+ intermediate state on disk might be wasteful
- Spark
	+ chained by operators
	+ data locality can be maximized due to entire data flow described
	+ parallel computation
	+ intermediate state materialized by command if necessary

## EP16: streaming processing
- unbounded data set
- producer, consumer, broker
- message broker
	+ delivery patterns: fan out (consumer group); load balancing (consumers in one group)
	+ types: in mem (redis); log base (kafka)
- use cases
	+ logging and metrics
		* windowing for accumulation
	+ change data capture: db change logs
		* can be compacted by only holding the most recent value of keys
	+ event sourcing
		* keep all the original events, can not be compacted
		* many different views can be derived from it, in eventual consistent way
- stream joins
	+ stream-stream join
		* window and watermark
	+ stream-table join
		* cache table in stream processor, with delta changes to update
	+ table-table join
		* batch data + delta changes
- fault tolerance / exactly once
	+ at least once: checkpoint, micro-batch
	+ idempotency / atomic transaction (2PC)

## EP17: consistent hashing
- sharding, load balancing
- ring of hash result range, multi sub-partitions of a node scattered on the ring
- dynamic add / remove node, less data set moved

## EP18: gossip protocol
- relatively low overhead to broadcast messages, for failure detection in large clusters
- broadcast message to few nodes randomly
- in cassandra
	+ each node gossip with other nodes regarding its own local load
	+ each node gossip with other nodes about heartbeats it receives from all other nodes, with a timestamp
	+ other nodes keep only the most recent messages (with the most recent timestamp) and pass them on
	+ all nodes use the most recent heartbeat timestamp to determine which nodes are still alive

## EP19: Cassandra deep dive
- NoSQL, wide column, column family
	+ row oriented storage
	+ rows can have any number and type of columns
	+ with one primary key as partition key
	+ other columns of a row can be designated as clustering keys
- storage: LSM tree + SSTable
	+ fast write
	+ slower read
- replication
	+ admin chooses quorum of write success, which could be non strong consistent
	+ conflict writes can be handled by
		* last write wins
		* read repair on reads
		* anti-entropy process in background
- fault tolerance
	+ data stored in replicas: read/write throughput for linear scalability; replication topology to tolerate DC level node invalid
	+ gossip protocol for heartbeat detection
- index: use clustering keys within a partition
	+ local index, not global index
	+ multi ordered clustering keys keeps multi copies, which slows down writes
- use case
	+ write heavy applications
	+ reads within a single partition
- pitfalls
	+ lack of strong consistency
	+ can not support data relationships
	+ lack of global secondary index

## EP20: coordination services
- write strong consistency of shared state, writes are slower than no broadcasting mechanism
- it is always for read heavy workloads, but not very good read performance
- by default, coordination service is not strong consistent for read
	+ no monotonic read, but monotonic read can be achieved if:
		* when client read/write, it stores the last seen offset
		* then it reads from a replica with the offset, it can judge if it is up to date or not, need to wait or try another replica
- ensure predicate validity: it is an alternative of predicate lock of db
	+ user can watch a key before read, and any related writes to db will update the key first, so the client can receive a notification that the read is out of date
	+ it is kind of a similar approach to serializable snapshot isolation, in a third-party coordination service
- for read strong consistency
	+ read from leader: leader traffic pressure
	+ sync
		* when client reads, it firstly writes a "sync" into replicated log, get the current offset, or writes the "sync" id into state
		* then client reads from any replica with the offset or "sync" id, it responds only if it is up to date: offset exceeds, or the "sync" id received
	+ quorum read: not completely perfect
		* read from majority of nodes, then at least one node has the most up to date write
		* corner case: an entry is committed in leader, but not yet committed in followers, then concurrent reads from majority nodes could get different results, which is not strong consistent; the same entry replicated to replicas can not be ready for read at the same time
- coordination service uses a centralized, replicated key value store built on top of a consensus algorithm
	+ it is not built to handle strong consistency out of the box, there are some ways of achieving strong consistent reads, at the cost of significantly reduced performance

## EP21: Hadoop file system design
- write a file once, read it many times
- files are stored in chunks, typically around 64-256mb per chunk
- chunks are replicated for data durability and availability
- name node: metadata of files, track of chunks
	+ all metadata in memory
	+ all changes write edit log (WAL)
	+ checkpoint of state for failure recovery
	+ receive block report from data nodes, automatically replicate insufficient replicas
- replication
	+ "rack aware" replication
	+ replication pipelined from one replica to the next
- hadoop reads
	+ name node tells the list of data nodes holding a chunk
	+ client reads from the proper data node, usually the closest one
- hadoop writes, to append a file
	+ pick a primary replica of the file chunk
	+ all other replicas are secondary, appended in the replication pipeline
	+ once all replicas acks the write, client receives a success
- HA HDFS
	+ active name node writes the edit log into zk (quorum journal manager)
	+ back up name node reads the log from zk, to rebuild state

## EP22: HBase/BigTable deep dive
- wide column, column family
	+ column oriented storage
- master server
	+ file metadata
	+ leverage zk for WAL and support a back up master node, for fault tolerance
- region server
	+ heartbeat to zk for aliveness report
	+ LSM tree + SSTable (HFile), in column oriented format
- replication
	+ region node is usually running on HDFS data node, it uses HDFS replication pipeline to synchronously replicate HFile
	+ column oriented storage provide high read throughput
- similar to Cassandra, both using LSM tree + SSTable, with difference that
	+ Cassandra: row oriented storage
	+ HBase: column oriented storage
	+ thus HBase is better for analytic queries

## EP23: conflict-free replicated data types (CRDT)
- multileader/leaderless replication increases write throughput, which inevitably leads to conflicts
- the goal of CRDT is convergence
- use cases
	+ collaborative editing
	+ online chat system
	+ offline editing
	+ distributed leaderless key-value store like Riak and Redis
- CRDT types
	+ operations based CRDT, commutative replicated data types (CmRDT)
		* only delta ops are transmitted
		* not idempotent, network ensures
	+ state based CRDT, convergent replicated data types (CvRDT)
		* should be commutative and idempotent
		* works well with gossip protocol

## EP24: Riak explained
- distributed key-value store, optimized for great write and read
- similar to Cassandra, it achieves high throughput by
	+ parititioning
	+ multi-leader replication
	+ LSM tree
	+ read repair + anti-entropy
- differences from Cassandra
	+ data modeling
		* key-value store
		* secondary index for queries other than key requires extra metadata, but it is not ideal
	+ conflict resolution
		* Cassandra uses last write wins, which loses data and is based on unreliable timestamp
		* Riak uses version vector, with sibling values of concurrent writes, application can handle the sibling values with its own logic for next write
		* Riak also support some CRDTs: counter, set, map

## EP25: distributed caching primer
- faster reads to the end client
	+ store precomputed values
	+ fewer network calls to database
	+ fewer workload on database
- increased complexity on writes
	+ write around: write db, then invalidate key in cache
	+ write through: write db, then write cache
	+ write back: eventual consistency
- local cache vs. global cache
	+ local cache: no extra network calls if cache hit
	+ global cache: independent scalability, better for replication and partitioning, globally used

## EP26: Redis and Memcached explained
- Memcached
	+ string to string hashmap
	+ consistent hashing
	+ compare and set
	+ no failure handling or replication
- Redis
	+ string to multi data types hashmap
	+ transactions on a single partition
	+ range query on a single partition
	+ disk persistence via checkpoint or WAL
- Redis cluster
	+ single leader replication with automatic failover
	+ gossip protocol with heartbeat among nodes
	+ 16384 hash ranges predefined, which can be moved between nodes
- why not use Redis cluster?
	+ strong consistency needed
	+ other replication patterns like multi-leader or leaderless
	+ prefer a coordination service for configuration or partition management other than gossip

## EP27: search indexes
- Lucene
	+ the most popular open source search index
	+ the index behind ElasticSearch and Solr
- Lucene architecture
	+ LSM tree + SSTable
		* write to in mem buffer first, can not be read yet, need to be on disk
		* eventually write to immutable SSTable index file on disk
		* SSTables are eventually merged and compacted
		* read from multiple SSTables, and results from them are eventually merged
	+ tokenization
		* when a document (text string) is written, it is split into terms
		* all problems of natural language processing should be covered, like handling punctuation, case, contraction, common words like "the" or "and"
	+ inverted index
		* after tokenization, each document is given an ID, and added into inverted index
		* inverted index is a map from terms (words) to list of document IDs, and sorted by term
		* space saved by using document IDs in the index
	+ other copies of indexes can be created for other search capabilities, like suffix search
	+ Lucene supports a lot of cool search capabilities: text, number, similar words, geolocation
- ElasticSearch
	+ Lucene index + distributed cluster
	+ it creates a bunch of local inverted indexes for the documents on a given node
	+ it is better to keep search on the same shard, and avoid across shard queries
- ElasticSearch caching
	+ cache of index pages in mem by operating system
	+ ES caches queries on a shard level in mem, with search result, parts of certain queries which might be used by other queries with the same filters
	+ assumption is the data on the shard has not changed

## EP28: time series databases
- write once, read many times, corresponding to a range of time
- InfluxDB, TimescaleDB, Apache Druid
- write optimize
	+ index of source id + timestamp; same data source of similar intervals of time can be on the same node
- read optimize
	+ column oriented storage; columns are usually similar and infrequently changed
	+ chunk table of (source id, time interval), simpler to maintain and keep index respectively; it is also easiler for truncation

## EP29: geospatial indexes
- geohashing
	+ split a region into subregions, and so on
	+ subregion hash code string prefixed by parent hash code
- geospatial index
	+ convert point to geohash
	+ find the geohash depth by distance size
	+ range query by geohash string
- geospatial index can be distributed over multiple nodes by partitioning
- Lyft uses Redis geospatial index; Uber build its own geospatial index called H3, which uses hexagon instead of rectangle
- similar index R-tree, for polygon shapes, not only for points

## EP30: chain replication
- chain replication
	+ write to head node
	+ read from tail node
	+ sync from head to tail, ack from tail to head
	+ strong consistency
	+ fault tolerance depends on external coordination service for node failure detection
- CRAQ: chain replication with apportioned queries
	+ if read(k) goes to a replica with highest version number of key k is clean, it safely return the value
	+ if read(k) goes to a replica with highest version number of key k is dirty, it must first ask the tail what is the current version number of the key, then return the version of value
	+ the read throughput is improved
- chain replication can be used for object storage

## EP31: content delivery networks (CDN)
- storing static content, object store
- world wide client requests needs CDNs
- geographically distributed cache for static content (rarely changed)
	+ push: servers periodically send data to CDN, e.g. news site, HTML home page
	+ pull: clients ask CDN for content, if not exist, it reads from server and store, e.g. social media site
- pros
	+ greately reduces load on server for popular content
	+ greately increase speed to users around the world
- cons
	+ may occasionally serve stale data, or slow down requests on cache misses
	+ have to change static URLs in database to reflect that they are now stored by the CDN

## EP32: Google Spanner
- Spanner is a database created by Google that falls into the category of NewSQL, it designs based on relational data model, with system scalability supported
	+ Relies on using synchronized clocks aka TrueTime (atomic clock and GPS)
	+ CockroachDB is similar, but using less synchronized clock (HLC with NTP), makes the solution a bit less viable
- Spanner guarantees
	+ ACID transactions
		* atomicity: 2PC across shards
		* serializability: 2PL on a single node
	+ replicas are consistent via a consensus algorithm
	+ linearizable reads and writes
- Spanner optimizes 2PL
	+ issue of 2PL: dead lock if 2 threads holds exclusive/write lock and shared/read lock respectively; Spanner doesn't require shared lock for reading threads, then performance enhanced, and dead lock avoided
	+ consistent snapshot: if a snapshot contains a write, then all the writes it depends on must be present, which keeps the causality
	+ Spanner uses TrueTime to track causality, only the writes with lower timestamp can be read
	+ TrueTime is expressed as a time range with very short interval `[t_min, t_max]`, which means the actual timestamp falls in the range with a very high probability, if `a.t_max < b.t_min`, then `a < b`, which means a happens before b
	+ a write at `[t1-d, t1+d]` must wait for `2d` before committing, it can be seen by a read at `[t2-d, t2+d]` only if `t1+d < t2-d`, so the snapshot seen by reader is consistent.
		* it is similar to the late messages in streaming system, with a watermark to determine which messages can be seen
		* the difference is that message could be late in streaming system, but here the timestamp is created in server node, so no late request due to network transmission, only by time drift, and TrueTime guarantees the time drift is very short
		* in TrueTime, `d = ~7ms`, `2d = ~14ms`, so the wait time of writes is acceptable, and performance not hurt
- Why not Lamport timestamp?
	+ Lamport timestamp provides a total ordering of writes with causality, but the ordering of concurrent writes could be unexpected, which is not linearizable
	+ Lamport timestamp in different nodes could have huge gap if no frequent cross traffic, so the ordering of different nodes is arbitrary, even an hour earlier write A in node 1 is ordered after current write B in node 2; then a reader can see `A` some minutes ago, but see `B < A` now, it is not consistent, and breaks linearizability
- Why not centralized timestamp?
	+ a centralized timestamp provider is called TSO (timestamp oracle), it gives timestamp/ordering to each write
	+ it will be a bottleneck of performance and single node of failure, also network latency for users around the world
- conclusion
	+ Spanner uses non-commodity hardware to achieve high throughput and linearizability, which is hard to be used for most businesses.
	+ if we can achieve synchronized clock, we can depend on the timestamp for ordering, even in distributed system.

## EP33: load balancing
- load balancer is a type of reverse proxy
- keep state of server node health via heartbeat, process at local as opposed to in a coordination service to speed up routing
- routing policies
	+ least connections
	+ least response time
	+ round robin, optional weighted
	+ hashing
- layer 4 vs. layer 7 load balancing
	+ layer 4: network transport layer, IP address, ports in header; compute faster
	+ layer 7: application layer, header contents; more flexibility
- consistent hashing, useful for cache, thus reduce amount of network calls
- availability, single point of failure is unacceptable, can be deployed as active-active or active-passive mode
- in reality, it often performs more functionalities than just load balancing
	+ encryption/decryption
	+ reverse proxy

## EP34: Amazon Aurora
- AWS Aurora is a NewSQL database which claims to gain 35x performance boost over traditional MySQL in cloud
	+ traditional RDBMs stores B-tree in disk, and page cache in memory
	+ Aurora decouples the cache and disk to two different servers, with log entries consumed, disk server derive state locally
	+ memory cache server and disk storage server can replicate independently
- quorum write; read from up to date replica with valid log position
- replication of caching layer: read-only cache replicas can be added, leader node asynchronously propagates log changes
- failure of database node
	+ when a leader node is elected, a quorum read is used to figure out the greatest committed log position, and the uncommitted ones are reverted
	+ when a server holding too much (like 1TB) storage fails, it is likely to be holding too many partitions; in order to avoid massive network overhead of copying, the shards on the node are sent to different nodes to parallelize the process of copying data

## EP35: MongoDB
- MongoDB is a document database, no enforced schema, but advised to be similar
- architecture overview
	+ within a collection, the documents are partitioned by shard key per record
	+ within a shard, availability is ensured via single leader replication
	+ B-tree based storage engine, read prioritizes write
- cross partition operations
	+ multi document transactions, either over the same or different shards
	+ parallelized queires over the entire data set, speed up via secondary index
- advantages
	+ flexible schema
	+ data locality by nested document structure
	+ secondary index supported, over multiple shards
	+ good for read by B-tree storage
- disadvantages
	+ lack of normalized data may require changes in multiple places
	+ single leader replication limits write throughput scalability than leaderless replication like Cassandra

## EP36: VoltDB
- VoltDB is a NewSQL database with a variety of optimizations in order to gain performance increases over traditional relational databases
- VoltDB executes in single thread, instead of 2PL for transaction serializability
	+ all data stored in memory: command log + snapshot
	+ transactions must first be stored as stored procedures: occurs in read modify write transaction
- partition & replication
	+ transaction across multiple shards uses one of the nodes as coordinator
	+ replication can be done both in single leader or multi-leader manner, conflicts can be resolved by operation priority or LWW strategy

## EP37: Monoliths vs. Microservices
- monoliths
	+ easy to implement, test, deploy, scale
	+ hard to onboard new dev, coorparate, infra upgrade
- microservices
	+ deploy and scale independently
	+ complexity of application layout
- docker
	+ container of applications
	+ preconfigure the capacity of virtual node

## EP38: Long Polling, WebSockets, Server Sent Events
- How to provide real time updates from server back to a client?
- polling: server overloaded
- long polling, longer time out than normal polling
	+ easy to implement, universal support
	+ one direction, excess connections on server, race condition of same client
- websocket, fully bidirectional channel, each registered to a port
	+ bidirectional communication in real time, lower network overhead
	+ less support on old browsers, overkill for infrequent change data
- server sent events, client registers for events from server
	+ no constantly established connection
	+ one direction, server overload if too many registered concurrent connections, complex implementation

## EP39: Apache Spark
- problems of MapReduce
	+ tons of disk IO due to materializing intermediate state to HDFS
	+ many redundant maps when not actually necessary
- Resilient Distributed Dataset (RDD)
	+ in memory
	+ can create from data on disk or by operation of another RDD
	+ keeps track of the lineage of each RDD
- fault tolerance
	+ narrow dependency: map operation
	+ wide dependency: reduce operation, checkpoint mechanism

## EP40: Apache Flink
- state in stream processing
- fault tolerance
	+ checkpoint in durable storage
	+ checkpoint barrier message in the message queue
- job manager node
	+ parallelize a job into work for each task manager using dataflow graph
	+ initiate checkpoints, and maintain the locations
	+ achieve HA by a coordination service like ZK or etcd, track node heartbeat
- log based message brokers as data source
- change cluster size
	+ checkpoint state is a list of key-value pairs, easy for re-partition

## EP41: bloom filter
- algorithm for data existance approximation
	+ each record hash value mod length, update the bit array
	+ when read, check data hash value existance
- false positive, no false negative
- to decrease false positive rate, reset a bloom filter, at the cost of losing historical data

## EP42: Merkle Trees
- algorithm for detecting difference between data sets or files, with logarithmic time cost
	+ each file has a hash value, as leaf
	+ several hash values grouped to a parent value, as root, the tree built
	+ change of leaf will impact the change of related root, comparison is easy to position
- used cases
	+ git diff
	+ cassandra anti-entropy, data consistency of multi-master replication

## EP43: data serialization (Protocol Buffer, Thrift, Avro)
- data serialization across systems
- Thrift & Protocol Buffer
	+ use a schema to greatly reduce the size of serialized messages
	+ field tag number to avoid field name
- schema evolution
	+ backward compatibility: new code can read old data
	+ forward compatibility: old code can read new data
	+ field tag can not change
	+ hard to change data type of field
- Avro
	+ schema used, but no field tag
	+ use schema to decode data, must be in the same order as the written fields
	+ schema revolution supported
		* fields matched up by field name in writer and reader schema
		* schema version number of writer scheam attached with each record
		* schema registry manages all the history schemas

## EP44: Apache Parquet
- Parquet chooses to use a hybrid approch, instead of pure row/column oriented
	+ partition the row space, and within each partition use column oriented storage
- data split into Parquet files (row groups)
	+ row groups are partitions of rows
	+ a footer with some metadata pertaining to the information in the row group
- row group is split into chunks (by column)
	+ each chunk of a given column contains 1MB pages, each page contains some metadata of the data in this page
		* min, max, count
		* dictionary of elements
		* bloom filter
- data encoding
- metadata accelerates queries, pre-sorted data can skip some pages

## EP45: TCP versus UDP
- Transmission Control Protocol (TCP), provides reliable, ordered (via seq num), and error checked (via checksum) streams of bytes
	+ handshake, three-way handshake: A --(seq x)--> B, B --(ack x+1, seq y)--> A, A --(ack y+1)--> B
	+ reliable transmission
		* dup ack retransmission: A --(data packet, seq 99)--> B
			- B --(ack, seq 100)--> A, success, will send next packet
			- B --(ack, seq 99)--> A, packet 99 missed, will resend packet 99 again
		* timeout based estimate, resend if ack not received within timeout
	+ flow control
		* rest buffer space of receiver is acked to sender, so sender will not send too much data later
		* if receiver has no room left, sender waits for some time, and sends a small packet to check space
	+ congestion control
		* network could be overwhelmed if too many packets sent over it
		* mitigation strategy: congestion window tracks number of packets of a connection, window size increase linearly (`+1`), but decrease exponentially (`/2`) if network overloaded
	+ connection termination
		* four-way handwave
			- A --(FIN)--> B, A active close, notify B
			- B --(ACK)--> A, B passive close, ack A
			- B --(FIN)--> A, B ready to close, notify A
			- A --(ACK)--> B, A ack B, and wait some time to close; wait time to ensure undelivered message can be discarded, in case if another new connection of this port can see them
- User Datagram Protocol (UDP), one node indiscriminately starts sending data to another node without any consideration for flow control, dropped packets, out of order, error detection, or congestion control
- comparison
	+ UDP: faster, no need for ack, just send
		* real time app that do not care data loss, like video chatting, video games, DNS server, stock price
	+ TPC: slower, guarantees of data delivery and ordering, network overload protection
		* better for most typical REST servers
		* provide guarantees against DDOS attacks

## EP46: Certificate Transparency (SSL/TLS)
- certificate transparency is used in HTTPS, which takes traditional HTTP request and adds an additional security layer (SSL/TLS)
- RSA (public key) encryption
	+ server generates public key and private key, public key is broadcasted to clients
	+ client encrypts message using public key
	+ server decrypts message using private key
	+ client has to make sure the public key is from the server, not a man in the middle
- Certificate
	+ verify that a server is actually who it says it is
	+ server must register a Certificate Authority, providing information about the server such as its owner, IP address, public key, authority it registered with (verifiable via a digital signature)
- Certificate Authority (CA)
	+ private key encrypt the certificate, it is used as the digital signature, client can decrypt the signature with public key
	+ if CA is hacked, can we trust it?
- [Certificate Transparency](https://certificate.transparency.dev/howctworks/)
	1. website owner requests a certificate from CA
	2. CA issues a precertificate
	3. CA sends precertificates to append-only logs
	4. precertificates are added to logs, with check of Merkle trees to make sure log is auditable
	5. logs return SCTs (signed certificate timestamps) to the CA, CA attach SCTs to a certificate
	6. CA sends the certificate to the domain owner, through a TLS handshake
	7. browsers and user agents help keep the we secure, by keeping the certificates
	8. monitor process periodically polls logs and report fake certificates, and newly issued certificates will be notified to domain owner
- CT is not perfect
	+ the cryptographic proof can not be done instantly
	+ it is possible that a malicious certificate authority can steal passwords for at least some small period of time until a monitor catches it

## EP47: Collaborative Text Editing
- OT (operational transform)
	+ transform function converts ordered operations into proper ones, so each local copy of the documents converges in the way it is supposed to
	+ the transform function could be very complicated with lots of edge cases
- CRDT (conflict-free replicated data type)
	+ operation based CRDT is better for text editing
	+ need to keep the intention of each collaborator
	+ requires causal broadcast for deletes
	+ later investigation of CRDT for rich text editing ([Peritext](https://www.inkandswitch.com/peritext/static/cscw-publication.pdf))

## EP48: Bitcoin
- decentralized currency, leverages blockchain, along with cyptographic proofs
- coin
	+ owner's public key
	+ hash of previous block
	+ previous owner's signature
- double spending problem, needs to be resolved by blockchain
- block
	+ hash of previous block
	+ transactions
	+ a nonce, to prove validity of the block
- a node wants to add a block, it must broadcast it to all the other nodes
	+ a peer node receives new transactions and adds them into block
	+ it uses some amount of CPU power to determine the nonce (proof of work)
	+ find x such that hash(transactions, prev_block_hash, x) starts with n leading zeros
	+ after nonce is found, the block is broadcasted to other peers
- once peers receive a block, they need to validate the block
	+ hash of the new block has the proper number of leader zeros
	+ prev block hash points to existing block in chain
	+ transactions in the block are valid
- forks in the blockchain
	+ if two peers find the noonce of a block at the same time, so some nodes process them in different orders
	+ or if one malicious peer sends a block to a subset of nodes and a different block to another subset
- peers will continue to add blocks to whichever branch of the fork that they received first; if it sees another branch has gotten longer it will abandon its current shorter branch and use the other branch.
	+ in this way, double spending is technically possible, but most parties will wait until a few blocks have been processed after the one holding a given transaciton in order to ensure that it is likely the transation will stay in the blockchain

## EP49: Neo4j (Graph Databases)
- data format
	+ node: point
	+ property: key value pair on node
	+ relationship: edge
	+ label: information to edge
- Neo4j is a native graph database, by representing data in its actual graph form on disk, with a significant performance improvement when traversing graph
- different data formats are stored separately, e.g. different files for nodes, relationships, properties, and labels
	+ each node contains a pointer to its first relationship
	+ each relationship contains a pointer to the next relationship from that node
	+ this works effectively as a mini index, data is linked, this is known as index-free adjacency
- by using index-freee adjacency, the speed of a query depends on the number of nodes involved, not the size of the whole graph
- challenges with graph data
	+ ACID: modifying a relationship is not an isolated operation; Neo4j has been forced to use a write ahead log for atomicity, as well as locking in order to ensure two transactions do not get jumbled up
	+ distributed transaction ordering: if two transcations overlap on multiple partitions, and involved partitions need to order them the same way; Neo4j tries to figure out the overlapped partitions, and use one of them as a sort of transction coordinator to ensure all involved partitions commit at the same time; for replication, Neo4j uses Raft to ensure consistency

## EP50: Approaching Interview Questions
- requirement clarification
	+ scale, performance goals
	+ functional requirement
- capacity estimation
	+ ratio of reads to writes
	+ number of users, average size of data
- API definition
	+ function calls
- data tables
	+ columns
	+ index
	+ sql
- get designing
	+ no single point of failure
	+ load balancing
	+ replication
	+ sharding
	+ database choice
	+ microservice
- why is prior to how
	+ list tradeoffs of design choices


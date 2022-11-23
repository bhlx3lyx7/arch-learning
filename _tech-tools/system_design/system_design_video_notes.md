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
	+ read repaire: read from multi nodes in parallel, take the most up to date piece and propagate it to the other replicas which had outdated data
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
- lost update: concurrent txns to read value, change and update, there might be update lost
	+ atomic write operation: CAS, exclusive lock
	+ explicit lock: hard to reason about, lead to many bugs
	+ automatic db detection: use snapshot isolation to detect lost updates (can detect the value it is about to write has been changed by other txn id), and then rollback and retry
	+ these solutions doesn't work in multi-leader/leaderless replication, because they assume one copy of data, better to store conflicts as siblings or use custom resolution logic
- write skew: concurrent txns to read a set of rows, make decision and update, which might impact the read set of rows
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



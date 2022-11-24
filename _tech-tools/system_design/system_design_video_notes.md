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
		* also possible, the follower's response may tell that it has a higher term number write, then the leader gives up being a leader
- comparing with 2PC, Raft is good for making replicated logs, but not so great for cross partition atomic transactions

## EP15: batch processing






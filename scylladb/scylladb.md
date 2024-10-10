# ScyllaDB

## Architecture

### Server arch
- cluster: whole cluster of nodes
- node: physical node; one node holds multiple vNodes, as physical distribution
- shard: dedicated to resources (CPU, memory, disk), aka "shared nothing" design; logical distribution, with several vNodes or tablets

### Data arch
- data model: "wide column" as Cassandra, with "key-key-value" format, partition key + clustering key + value
- keyspace: top level container of data, with isolation, replication strategy, replication factor, etc.
- table: within a keyspace, data is stored in separate tables; only standalone table supported, no join operation
- partition: logical partition of data set, row-oriented, same as shard, implemented as vNode or tablet
- row: record rows are sorted in a particular order
- column: each column can define its data type, encryption, some columns are used to define index and sorting, known as partition and clustering keys
- cell: row + column = cell, of any data type, even user defined datatypes

### Ring arch
- ring: ring of token ranges, each partition mapped to a single hashed token (one token can hold several partitions), for data distribution
- vNode: ring is broken into vNodes, which holds a range of tokens, assigned to a physical node or shard; static partition
- tablet: similar to vNode, but can be dynamically re-partitioned to avoid hot partition; by default, each tablet is 5GB

### Storage arch
- memtable: in write path, data is first put into a memtable stored in RAM, for built-in cache
- commit log: append-only log of local node operations, written at the same time when data sent to memtable, for failure recovery
- SSTable: persist on disk, immutable, update or delete operation only record in new SSTable, will be compacted periodically
	+ tombstone: mark as delete, record is removed when compacted
	+ compaction: periodic compaction of SSTables to free up disk space
	+ compaction strategy: determine when and how best to run compactions, by trade-offs between write, read, and space amplification

#### Read path
when read a record, 
1. if cache hits, return result; otherwise read from disk
2. read SSTable metadata (in mem), find target SSTables by partition key
3. bloom filter to check if exists
4. read from memtable (in mem) or SSTable, if found, write to cache

#### Write path
when write a record,
1. write to memtable (in mem), simultaneously write to commit log
2. memtable is flushed to new immutable SSTable in batch

### Client-Server arch
- driver: ScyllaDB communicates with applications via drivers for its Cassandra-compatible CQL interface, or with SDKs for its DynamoDB API

## System properties

### peer-to-peer arch
- Raft cluster for system metadata storage, like topology change of partitions, schema change, etc.
- automatic data replication between replicas
- tunable consistency, configurable consistent level, like one replica, quorum, or all replicas

### zero downtime
- rack and datacenter awareness, topology aware, knowing which rack and data center a node belongs to, replicas can be spread across different data centers, to tolerate disasters
- multi-datacenter replication, different data center can have different replication factor (RF)

### anti-entropy
- temporary node failure can be recovered by commit log
- hinted handoffs: temporary node outage (less than 3 hours) can use hinted handoffs to keep track of what transactions occurred while the node was unavailable, and catch up when it comes back
- row-level repair: background repair process to sync up data with other replicas

## Reference
- ScyllaDB: https://www.scylladb.com/product/technology/
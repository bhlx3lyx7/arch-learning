# Distributed Lock

## Redis

- single Redis node
- lock: set if not exists
- release: compare lock value and delete

Not safe, but good performance, 300K/s maybe

## Redlock

- multi Redis nodes
- lock: set if not exists, to all nodes, success if most nodes respond success

Better safty (not safe at all), but still good performance, 100K/s

In the critiques from Martin Kleppmann, there are two points:
1. Redlock is built on a timestamp-based system model, and it is difficult for multiple servers to guarantee consistent timing, which makes the actual expiration times of locks different.
2. distributed locks with automatic expiration must provide some fencing token (uniqueness constraint) mechanism, such as monotonically increasing IDs, to guarantee mutual exclusivity over shared resources, and Redlock does not provide such a mechanism.

### My opinion

1. For the first point, I agree with Martin that timestamp is not trustable in the distributed world.
2. For the second point, I agree with Antirez that the fencing token doesn't solve the problem, actually it is a common problem of all the distributed locks. The only way to resolve it depends on other solutions like:
	- idempotency of the write operation, from the biz perspective
	- consensus-based storage to deduplicate the token (unnecessary incremental)

Overall, distributed lock is only used to avoid concurrency in MOST of the time, the strong guarantee can only be provided by consensus-based storage, or idempotent work-around in biz workflow.

## etcd distributed lock

- lease
- TTL
- heartbeat
- advanced mechanisms
	+ unified prefix, each client sets a key with same prefix, e.g. `/lock/xfew123x`, `/lock/ewr216xa`
	+ revision, increment for each write operation, used to determine the order to occupy the lock
	+ watch, each client watches the previous key, get notified if watched key changes

More safe, with lower performance, 50K/s in performance test, but in prod, maybe 10K/s

## Chubby

- coarse-grained locks
	+ long time locks, not for short time locks
	+ traffic pattern: low frequent acquire, high frequent heartbeat
- Paxos consensus
- write/read via master node, for linearizable (strong) consistency
- lock mode: exclusive, shared
- sequencer (similar to fencing token), storage can check lock in two ways:
	+ call Chubby API `checkSequencer()` to validate the lock is valid or not 
	+ compare the client sequence with the latest sequence, and deny if the client sequence is smaller
- the two ways are intrusive to storage, so Chubby has a lock-delay mechanism:
	+ client requests a lock-delay time for its lock, when Chubby finds the client is passively disconnected (no heartbeat), it does not release the lock, but still prevent other clients from acquiring it until the lock-delay time exceeds
	+ this lock-delay mechanism is just similar to the lease

Typical use case is for GFS (google file system), for cluster leader election, 93% traffic is keepAlive heartbeat, each Chubby cell node can serve 10K machines.

Chubby also provides low-volume storage, to act as repository of metadata or configuration of distributed system.

### Caching in Chubby

- Chubby client lib
	+ keeps long-live RPC with Chubby master node
	+ has a write-through cache
- Chubby master node
	+ notifies clients by events, control & data event
	+ maintains each client's cache contents
	+ when metadata changes, master blocks it first
	+ master notifies invalidation event to cached clients
	+ clients invalidate the cache content and ack to master
	+ after all related clients ack, master proceeds the the change

Invalidation is more efficient than update. Cache is useful if Chubby serves as a configuration service, and this mechanism even keeps client caches consistent.

### Failover

1. Chubby master fails, new master is elected
2. Chubby clients resume keepAlive RPC to new master, but no operation
3. new master sends a failover event to each client to flush cache
4. once a client acks the failover event, new master allows subsequent operations from this client

### Use cases

Chubby is used for
- GFS master election, and matadata storage
- Bigtable leader election, and metadata storage
- ACL files storage

### My opinion

- Chubby is better used as name service, configuration service, as well as distributed lock service for coarse-grained locks.
- The consistent caching mechanism is charming, and really useful for configuration of distributed system
- Distributed lock has its common problem, Chubby can not solve it, no matter fencing token, sequence, lease, etc.; so distributed lock can not guarantee complete security
- The double check mechanism in storage service might help in some way, because it is less likely that both the client and the storage service encounter the program stall; but it is inpractical for most storages

## Conclusion

- Distributed lock can not guarantee complete security
- better to choose for use cases
	+ for efficiency, like less (not none) concurrent work, Redis single node is enough, occasional failure is acceptable
	+ for correctness, like master election, zk/etcd/chubby solutions are better, but there might still be edge cases

## References
- Talking about distributed lock implementation: https://www.sobyte.net/post/2022-08/distributed-lock/
- How to do distributed locking: https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html
- Is Redlock safe? http://antirez.com/news/101
- The Chubby lock service for loosely-coupled distributed systems: https://research.google/pubs/the-chubby-lock-service-for-loosely-coupled-distributed-systems/
- The Chubby Lock Service: https://github.com/jguamie/system-design/blob/master/notes/chubby-lock-service.md
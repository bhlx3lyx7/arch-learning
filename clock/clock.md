# Clock in distributed system

Clock is not reliable in distributed system, so we have to find other ways to judge the order of concurrent requests.

## Different timestamps

### PT, physical time
the physical time of each node, would be drift between nodes

### LC, logical clock
Lamport timestamp is a type of logical clock, it keeps the causality relationship (happened-before) by message transmission, not by time pass

LC can achieve session consistency

### VC, vector clock
each event keeps a vector of nodes, to build a consistent snapshot, which is better than LC, but much more space required, especially for many nodes in cluster

### TT, true time
achieved by Google Spanner, using atomic clock and GPS, to make the time error range less than `7ms`, so only if two timestamps within `2*7=14ms` error window can be uncertain; the solution to resolve the error window is to wait for `14ms`, then the transactions can be ordered by timestamp

TT can achieve linearizability

### HT, hybrid time
PT + VC, VC only keeps the nodes with PT drift within error window, so its space cost is less

### HLC, hybrid logical clock
PT + LC, high digits store PT, low digits store LC; 

- CockroachDB uses HLC, can achieve linearizability within row, not multi rows
- HLC works based on NTP (Network Time Protocol, is an internet protocol used to synchronize with computer clock time sources in a network), with a larger error window than TT, for example `250ms`
- HLC will only read system time, but do not modify it; so if it drifts too much, HLC downgrades to LC; even the limited digits for LC can not handle the traffic, so practically it rejects and alerts
- the algorithm can refer to: https://www.jianshu.com/p/8500882ab38c

### TSO, timestamp oracle
centralized timestamp provider, to provide timestamp from a single source of truth

TSO can achieve linearizability

- a single node or a strong consistent cluster like etcd, providing HA but lower QPS; one node can bulk get timestamps at one time, so traffic is not a concern
- for better latency, it is not a global cluster
- TiDB uses TSO
- the design of TSO can refer to
	+ https://ipotato.me/article/67
	+ https://cloud.tencent.com/developer/article/1882195

*In TiDB, PD (placement driver) stores the metadata, and dispatch the universal unique ID for distributed transactions; the transaction ID is actually the TSO dispatched HLC (hybrid logical clock), having a global order*

## References
- 分布式事务中的时间戳: https://ericfu.me/timestamp-in-distributed-trans/
- HLC论文浅析: https://zhuanlan.zhihu.com/p/341293270
- 分布式系统的时间: https://www.jianshu.com/p/8500882ab38c
- 分布式系统中的时钟与一致性解读 1: https://ost.51cto.com/posts/15990
- 分布式系统中的时钟与一致性解读 2: https://ost.51cto.com/posts/15992
- Distributed Txn Without Centralized Sequencer/TSO: https://nan01ab.github.io/2021/05/Distributed-Txn(2).html
- PD 授时服务 TSO 设计简析: https://ipotato.me/article/67
- TIDB 的大脑 PD 到底是干什么的: https://cloud.tencent.com/developer/article/1882195


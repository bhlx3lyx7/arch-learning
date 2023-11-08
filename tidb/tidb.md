# TiDB

## Timestamp
- trxn ordered by timestamp
- each trxn has a start timestamp and commit timestamp
- each trxn reads from snapshot before its start timestamp, so another trxn can be seen only if its commit timestamp is before the start timestamp of current trxn
- Google Spanner uses waits `14ms` for each trxn, to guarantee all its previous trxns with smaller commit timestamp are all committed, to make sure all its seen snapshot will not change when it reads
- TiDB and CockroachDB uses the same timestamps and read/write lock to achieve SSI (Serializable Snapshot Isolation)

## References
- official site: https://docs.pingcap.com/zh/tidb/stable
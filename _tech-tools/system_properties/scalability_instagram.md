# Scaling of Instagram

## Instagram everyday
- 400 million users
- 4 billion likes
- 100 million photos/vedios
- top account: 110 million followers

## Scale out
problems:
- capacity
- power outage
- human mistake

need more data centers, so need to scale out applications

applications are categorized into:
- storage: stateful
- computing: stateless
- cache: high write/read, low latency

### storage
PostgreSQL
- leader write, follower read
- leader and followers in different DC
- latency increases across DC, with batch write to reduce sync times
- latency increase is acceptable, so it works

Cassandra
- replicas across DC
- 2 write, 1 read
- straight forward to scale out

### computing
- group django (UI server), rabbit MQ, celery in the same pod, within the same DC
- each DC has its own deployment

### cache
Memcache is also deployed in each data center.

challenge
- high performance key-value store in memory
- millions of write/read per second
- sensitive to network condition, so latency is a big deal
- cross region operation is prohibitive

solution
- write to storage (PostgreSQL) leader, sync to follower across DC
- in each DC, invalidate its own cache by storage nodes request
- users read from cache, invalid, then read from the storage node, to get the fresh value with best effort

some more challenges and solutions
- when hit DB, `select count(*) from table` costs 100+ ms. solution is to build a dedicated count table to record the counter, so it turns to `select count from table`, it costs 10+ us.
- when cache invalid, traffic pours to DB. solution is to use lease-get in memcache, the first user will get a `miss` and occupy a lease to access DB, the subsequent users will only get a `miss` and waiting for the lease released, or just use a stale value if acceptable. when the first user gets the fresh value, and lease-set into the cache, the latter queries will get a `hit`.

### benefits
- capacity
- reliability
- regional failure ready

## Scale up
problems:
- hardware growth is faster than user growth

need to enhance the ability of single hardware
- use as few CPU instructions as possible
- use as few servers as possible

### CPU instruction
- monitor: monitor metrics change with regression test
- analyze: tool to analyze performance of critical path
- optimize: find a proper solution to optimize and prove

### server num
memory
- share memory: 1 server n processes, to reduce shared memory of processes, use less servers
- reduce code: run in optimized mode, remove dead code
- share more: move configurations into shared memory, disable GC

network latency
- sync processing model with long latency
- async model helps to get fewer latency, and resolve the worker starvation problem

## Scale dev team
what we want:
- automatically handle cache
- define relations, no worry about implementations
- self service by product engineers
- infra focuses on scale

### data model
domain knowledge identified, define simplified data objects and relationships, as the basic knowledge across the teams, so no worry about basic things break down.
better data modeling and API increases developer velocity.

### source control
problems of multiple branches:
- context switching
- code sync/merge overhead
- surprises
- hard to refactor and major upgrade
- performance tracking harder

adopt one master approach, no branches
- continuous integration
- collaborate easily
- fast bisect and revert
- continuous performance monitoring

**feature test**
each feature will be onboarded gradually, from engineers to real users, to make sure the features runs as expected.

**load test**
load test of mock traffic helps to make sure the storage and cache can hold production traffic.

**velocity**
rollout once a diff, 40-60 rollouts per day.

**make sure no break down**
- code review, unit test
- code accepted, committed
- canary rollout
- to the wild
- monitoring and alert

## takeaways
scaling is
- continuous effort
- multi-dimensional
- everybody's responsibility
# Principles of Distributed Database System

## Chapter 1 - Introduction

We define a **distributed database** as a collection of multiple, logically interrelated databases located at the nodes of a distributed system.

### Data Delivery Alternatives (between sites)

- delivery modes: pull-only, push-only, and hybrid
- frequency: periodic, conditional, and ad-hoc
- communication method: unicast (one-to-one), one-to-many

### Promises of Distributed DBMSs

- transparent management of distributed and replicated data
- reliable access to data through distributed transactions
- improved performance
- easier system expansion

#### Transparent Management

- Data Independence
	+ logical data independence: user app decouple from change of data logical structure, e.g. schema
	+ physical data independence: user app decouple from change of data storage, e.g. expansion
- Network Transparency
	+ location transparency: independent of location of data and running system
	+ naming transparency: unique name is provide for each object in database
- Fragmentation Transparency: user app is independent of data partitioning
- Replication Transparency: user is not involved with handling copies

#### Reliable Access

- high availability to hide impact of partial failure
- full transaction support gurantees concurrent execution will not violate database consistency

#### Improved Performance

- distributed data fragments
	+ performs better by more resource of CPU and IO
	+ data locality reduces remote access delays
- parallelism of distributed system
	+ interquery parallelism: parallel execution of multiple queries of concurrent transactions
	+ intraquery parallelism: the same query is executed in different sites, for different portion of data
- interoperator parallelism
	+ pipeline parallelism: producer-consumer link operators can be executed in parallel, without materialize entire intermediate result, thus saving memory and disk resource
	+ independent parallelism: independent operators can be executed in parallel
- intraoperator parallelism: decompose one operator into independent suboperators
	+ each operator instance process one partition of data, proper initial partitioning of data set will frequently benefit the joins between data sets

#### Scalability

distributed system is much easier to accommodate increasing size and bigger workload.

### Design Issues

#### a. Distributed Database Design

One global databse, and the end result is a distribution of data across sites, this is referred to as *top-down design*.

data distribution
- partitioned (non-replicated): disjoint partitions plcaed at different sites
- replicated
	+ fully replicated: entire database is stored at each site
	+ partially replicated: each partition of database is stored at more than one site, but not at all sites

metadata distribution also faces a similar problem, needs to choose a proper solution

#### b. Distributed Data Control

- view management
- access control
- integrity enforcement

#### c. Distributed Query Processing

strategy to execute query over network in the most-effective way, need to consider these factors:
- distribution of data
- communication cost
- lack of sufficient locally available information

#### d. Distributed Concurrency Control

Solutions to achieve mutual consistency
- pessimistic
	+ synchronize executions before starts
	+ locking based approaches
	+ deadlock prevention, avoidance, detection, recovery
- optimistic
	+ execute and then check if any conflict
	+ timestamping based approaches

#### e. Reliability of Distributed DBMS

To achieve reliability, distributed DBMS should be able to
- detect & recover from failure
- ensure data consistency

#### f. Replication

Replication protocol should ensure the consistency of replicas, in eager way or lazy way

#### g. Parallel DBMSs

Parallel DBMS aims to high scalability and performance, which usually exist as parallel clusters in distributed DBMS

#### h. Database Integration

Multi-database systems can be integrated as a "looser" federation among data sources, which can be heterogeneous, this involves *bottom-up design*.

#### i. Alternative Distribution Approaches

different approaches for different network challenges
- peer-to-peer computing
- across sites

#### j. Big Data Processing and NoSQL

- volume: very high volume
- variety: multi-modal
- velocity: high speed as data stream
- veracity: quality concern due to uncertain source and conflict

### Distributed DBMS Architectures

Orthogonal dimensions of distributed DBMS architecture
- autonomy
- distribution
- heterogeneity

#### Autonomy

Autonomy refers to the distribution of control, not of data. It is a function of factors such as
- whether the component systems (individual DBMSs) exchange information
- whether they can independently execute transactions
- whether one is allowed to modify them

There are 3 alternatives:
- tight integration
- semi-autonomous
- total isolation

#### Distribution

Distribution refers to the distribution of data. 

There are 3 alternatives:
- non-distributed
- client/server
- peer-to-peer (full distribution)

#### Heterogeneity

Heterogeneity refers to the differences between component systems, such as hardware, network protocols, data models, query languages, transaction management protocols, etc.

There are 2 alternatives:
- homogeneity
- heterogeneity

### Client/Server Systems

- Server handles query requests, responds query results
- Client provides user interface, connects with server, and caches query results

### Peer-to-Peer Systems

- User processor handles user query requests, using GCS (global conceptual schema), logically access
- Data processor handles sub-query requests, using LCS (local conceptual schema), physically access

### Multi-database Systems

Individual DBMSs are fully autonomous and have no concept of cooperation, a dedicated mapping layer stands between the user and them.

Another model is mediator/wrapper architecture, wrappers stands in front of each DBMS, mediators sits in the middle layer to do the rest tasks.

### Cloud Computing

Pros:
- cost
- ease of access and use
- quality of service
- innovation
- elasticity

Cons:
- provider dependency
- loss of control
- security
- hidden costs to make apps cloud-ready

Multi-tenant database models:
- Shared DBMS server, tenants have different databases
- Shared database, tenants have different schemas and tables
- Shared tables, tenants have different records

## Chapter 2 - Distributed and Parallel Database Design

The distribution design starts from this global conceptual schema (GCS) and follows two tasks: partitioning (fragmentation) and allocation.

1. distribution design (need auxiliary information): GCS -> set of LCSs
2. allocation: set of LCSs -> each LCS
3. physical design: LCS -> physical schema

Reasons of fragmentation in distributed DBMS:
- data locality
- concurrent query execution: interquery and intraquery parallelism

Reasons of fragmentation in parallel DBMS:
- load balancing: interquery and intraquery parallelism

### Data Fragmentation

- horizontally fragmentation: interquery parallelism
- vertically fragmentation: intraquery parallelism
- hybrid fragmentation

Requirement of data fragmentation:
1. Completeness: no data loss
2. Reconstruction: sub sets can reconstruct to full set
3. Disjointness (for horizontally fragmentation): unique records

#### Horizontal Fragmentation

- primary horizontal fragmentation: partition by predicates defined in current relation
- derived horizontal fragmentation: partition by predicates defined in another relation

#### Vertical Fragmentation

It is huge number of possible combinations of columns, thus we have to resort to heuristic approaches for vertical fragmentation
- grouping: start by assigning each attribute to one fragment, and at each step, join/group some of them, until some criteria are satisfied; might lead to overlapping attributes
- splitting: start with a relation and decide on beneficial partitionings based on the access behavior of applications to the attributes; will not overlap

1. figure out attribute existance in queries
2. cluster algorithm to group related attributes
3. split algorithm to split attribute groups
4. check for correctness

#### Hybrid Fragmentation

In some special cases, vertical fragmentation and horizontal fragmentation may be followed by the other, producing a tree-structured partitioning

### Allocation

replication types:
- full replication
- partial replication
- partitioned without replication

partitioning techniques:
- workload-agnostic: round-robin, hash, range
- workload-aware: partition by query pattern, better used for vertical partitioning, related partitions can be replicated and located together

### Adaptive Approaches

- How to detect workload changes that require changes in the distribution design?
- How to determine which data items are going to be affected in the design?
- How to perform the changes in an efficient manner?

1. detect workload changes
2. detect affected items
3. incremental reconfiguration: instead of batch run of re-partitioning

## Chapter 3 - Distributed Data Control


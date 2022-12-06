# Software Architecture: The Hard Parts

Notes of the book reading.

## Chapter 1 - What Happens When There Are No “Best Practices”?

### ADR (Architecture Decision Record)
**ADR**: A short noun phrase containing the architecture decision

**Context**  
In this section of the ADR we will add a short one- or two-sentence description of the problem, and list the alternative solutions.

**Decision**  
In this section we will state the architecture decision and provide a detailed justification of the decision.

**Consequences**
In this section of the ADR we will describe any consequences after the decision is applied, and also discuss the trade-offs that were considered.

### Architecture fitness functions
Fitness functions to guarantee the implementation fits the architecture rules. For example,
- no cyclic dependencies between components
- layered architecture should avoid across layer dependency

The fitness functions should be tested like the unit tests, to make sure the rules not broken. The other checks can be done manually via a checklist.

## Chapter 2 - Discerning Coupling in Software Architecture

### Architecture quantum
- independently deployable
- high functional cohesion
- high static coupling: static coupling means dependency of like OS, frameworks, libs, etc. A single coupling point like a database of multi components, will lead to a single architecture quantum.
	+ service-based architecture relies on a single database, but multi services, which is slight different from microservices
	+ orchestrated event driven architecture has a coupling point of the orchestrator
	+ broker-style event driven architecture could has a coupling point of single database, and the indirect dependent components also actually couples with the database
	+ different modules with different databases like microservice are not a single architecture quantum
- dynamic quantum coupling: dynamic coupling means dependency of like sync or async communication at runtime
	+ communication: sync or async
	+ consistency: atomic or eventual
	+ coordination: orchestration or choreography
		* orchestration: a logical orchestrator controls the workflow
		* choreography: no central orchestrator, each component works in parallel, always event driven

*Chapter 12 will introduce the 8 saga patterns with trade-offs respectively*

## Chapter 3 - Architectural Modularity

### Modulariy Drivers
- Agility
	+ Maintainability
	+ Testability: While architectural modularity generally improves testability, it can sometimes lead to the same problems that exist with monolithic, single-deployment applications. If the communication between multi services is more, the test scope would cover all the related services, not only the single one.
	+ Deployability: *If your microservices must be deployed as a complete set in a specific order, please put them back in a monolith and save yourself some pain.*
- Speed to market
- Competitive advantage
	+ Scalability: Like testability and deployability, the more services communicate with one other to complete a single business transaction, the greater the negative impact on scalability and elasticity. For this reason, it is important to keep synchronous communication among services to a minimum when requiring high levels of scalability and elasticity.
	+ Availability / Fault tolerance: if other services are synchronously dependent on a service that is failing, fault tolerance is not achieved. This is one of the reasons asynchronous communication between services is essential for maintaining a good level of fault tolerance in a distributed system.

*Communication between components impacts the core system abilities.*

## Chapter 4 - Architectural Decomposition

### Is the codebase decomposible

- afferent and efferent coupling
	+ afferent (incoming) couplings: the number of classes in other packages that depend upon classes within the package is an indicator of the package's responsibility
	+ efferent (outgoing) couplings: the number of classes in other packages that the classes in a package depend upon is an indicator of the package's dependence on externalities
- abstractness and instability
	+ abstractness = num(abstract elements) / (num(abstract elements) + num(concret elements)), indicates the abstraction degree of package
	+ instability = efferent / (efferent + afferent), this metric is an indicator of the package's resilience to change
- distance from the main sequence
	+ distance = abs(abstractness + instability - 1), this metric is an indicator of the package's balance between abstractness and stability, ideal package is exactly on the main sequence line (A + I = 1)
	+ the upper-right corner enter into zone of uselessness, code is too abstract becomes difficult to use
	+ the lower-left corner enter into zone of pain, code with too much implementation and not enough abstraction becomes brittle and hard to maintain

reference [software package metrics](https://en.wikipedia.org/wiki/Software_package_metrics)

### Component-Based Decomposition

- When breaking monolithic applications into distributed architec‐ tures, build services from components, not individual classes.
- When migrating monolithic applications to microservices, con‐ sider moving to a service-based architecture first as a stepping- stone to microservices.

### Tactical Forking

The tactical forking pattern is a pragmatic approach to restructuring architectures that are basically big balls of mud.  

Fork another branch and delete is better than extraction.

## Chapter 5 - Component-Based Decomposition Patterns

1. Identify and Size Components
2. Gather Common Domain Components
3. Flatten Components
4. Determine Component Dependencies
5. Create Component Domains
6. Create Domain Services

## Chapter 6 - Pulling Apart Operational Data

Breaking apart a database is hard—much harder, in fact, than breaking apart applica‐ tion functionality.

### Data Decomposition Drivers

- Data Disintegrators
	+ change control
	+ connection management
	+ scalability
	+ fault tolerance
	+ architectural quanta
	+ database type optimization
- Data Integrators
	+ data relationships
	+ database transactions

### Decomposing Monolithic Data

1. Analyze Database and Create Data Domains
2. Assign Tables to Data Domains
3. Separate Database Connections to Data Domains
	- When data from other domains is needed, do not reach into their databases. Instead, access it using the service that owns the data domain.
4. Move Schemas to Separate Database Servers
5. Switch Over to Independent Database Servers

### Selecting a Database Type

Characteristics to consider when choosing a database:
- Ease-of-learning curve
- Ease of data modeling
- Scalability/throughput
- Availability/partition tolerance
- Consistency
- Programming language support, product maturity, SQL support, and community
- Read/write priority

Different database types:
- Relational Databases (e.g. PostgreSQL, Oracle): consistent, balanced read and write, easy to learn
- Key-Value Databases (e.g. Amazon DynamoDB, Redis): sclability, high throughput, high availability, better for read over write
- Document Databases (e.g. MongoDB, Couchbase): better modeling, good scalability and availability, better for read over write
- Column Family Databases (e.g. Cassandra, Scylla): scalability, high throughput, high availability, better for write over read
- Graph Databases (e.g. Neo4j, Infinite Graph): scalability, availability, consistency, better for read over write
- NewSQL Databases (provide the scalability of NoSQL databases while supporting the features of relational databases like ACID; e.g. VoltDB, NuoDB, TiDB): similar to relational database, with better scalability and availability
- Cloud Native Databases (e.g. Snowflake, Amazon Redshift): scalability, availability, consistency, better read over write
- Time-Series Databases (e.g. Influxdb, TimescaleDB): scalability, consistency, better read over write

## Chapter 7 - Service Granularity

### Granularity Disintegrators

- Service scope and function
	+ Single-purpose services with tight cohesion
- Code volatility
	+ Agility (reduced testing scope and deployment risk)
- Scalability and throughput
	+ Lower costs and faster responsiveness
- Fault tolerance
	+ Better overall uptime
- Security
	+ Better security access control to certain functions
- Extensibility
	+ Agility (ease of adding new functionality)

### Granularity Integrators

- Database transactions
	+ Data integrity and consistency
- Workflow and choreography
	+ fault tolerance of sync calls
	+ performance and responsiveness
	+ data consistency and integrity
- Shared Code: Maintainability
	+ Specific shared domain functionality
	+ Frequent shared code changes
	+ Defects that cannot be versioned
- Data Relationships
	+ Data integrity and correctness

### Finding the Right Balance

Make decision by balancing the reasons of disintegrators and integrators.

## Chapter 8 - Reuse Patterns



## Chapter 9 - Data Ownership and Distributed Transactions

### Distributed Transactions

ACID vs BASE: instead of ACID, distributed transactions support BASE.

### Eventual Consistency Patterns

- Background Synchronization Pattern: background application synchronize different data storages
	+ pros: Services are decoupled, Good responsiveness
	+ cons: Data source coupling, Complex implementation, Breaks bounded contexts, Business logic may be duplicated, Slow eventual consistency
- Orchestrated Request-Based Pattern: dedicated orchestrator service in charge of the transaction process
	+ pros: Services are decoupled, Timeliness of data consistency, Atomic business request
	+ cons: Slower responsiveness, Complex error handling, Usually requires compensating transactions
- Event-Based Pattern: different services are asynchonously triggered by event
	+ pros: Services are decoupled, Timeliness of data consistency, Fast responsiveness
	+ cons: Complex error handling


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

Breaking apart a database is hard—much harder, in fact, than breaking apart application functionality.

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

WET: Write every time or Write everything twice
DRY: Don't repeat yourself

No reuse in distributed architecture, but in practice there are still some reuse patterns.

### Code Replication

Advantages:
- preserves the bounded context
- no code sharing

Disadvantages:
- difficult to apply code changes
- code inconsistency across services
- no versionning capability across services

### Shared Library

granularity and versioning are the most important concern.

#### Dependency Management and Change Control
- smaller, functionally partitioned libraries is better
- favor change control over dependency management

#### Versioning Strategies
- always use versioning
- version deprecation is difficult
- version communication is difficult
- avoid use the LATEST version, which might be incompatible sometime

#### When to Use
- homogeneous environment
- shared code change is low
- performance, scalability, fault tolerance not impacted
- risk of breaking is low because of versioning

### Shared Service

shared code must be in the form of composition, not inheritance.

#### Change Risk
- changes are generally runtime, with more risk to services
- versioning can help reduce the risk, but it is more complex to apply and manage, due to different endpoints and protocols

#### Performance
- network and security latency introduced
- async messaging can help mitigate the issue

#### Scalability
- shared service must scale to serve if more traffic

#### Fault Tolerance
- if the shared service unavailable, clients will be impacted

#### When to Use
- polyglot environment, with multiple heterogeneous languages and platforms
- shared functionality tends to change often
- be careful of runtime side effects and risks

### Sidecars and Service Mesh

- duplication vs. coupling
	+ in microservice arch, duplication wins, because one of the design goals of microservice is a hight degree of decoupling
	+ but in service-based arch, coupling wins
- sidecar can be deployed with business service, to provide non-business functionalities, like monitoring, logging, etc.
- all nodes with sidecar component build up a service mesh, to provide the management of nodes in enterprise level
- orthogonal coupling means two distinct purposes must intersect to form a complete solution, like the business logic and monitoring
- it should consolidate operational coupling, not domain coupling

#### When to Use
- a clean way to spread some sort of cross-cutting concern across a distribtued architecture
- it is an architectural equivalent to the decorator design pattern, we can decorate bahaviors across a distributed architecture independent of the normal connectivity

#### About Reuse
Reuse has two important aspects:
- abstraction, highly abstraction finds common things to reuse
- rate of change, slow rate of change mitigates change risk

Organization builds a platform with a well-defined API to hide the implementation details
- benefits from the abstracted API
- hide the changes of implementation
- should carefully design the encapsulation and contracts

## Chapter 9 - Data Ownership and Distributed Transactions

### Assigning Data Ownership
- General rule, the service writes the data owns it
- But if multiple services write the same table makes joint ownership, even common ownership

### Single Ownership
only 1 service writes to a table, it owns the table

### Common Ownership
multiple services write to a table, leading to common ownership
- a dedicated service can be assigned to own the table
- if no sync reply required, a message queue can be placed

### Joint Ownership
several services write to a table, leading to joint ownership

#### Table Split Technique
Split 1 table to more, each service owns its own table; but need consider trade-off of consistency and availability, sync or async communication

#### Data Domain Technique
A dedicated domain scope of the shared table; change of data schema would impact the services in other domains

#### Delegate Technique
One service is assigned single ownership of the table and becomes the delegate, other services communicate with the delegate; but introduces high level of service coupling, and some harm to non-owner writes, like low performance, no atomic transaction, and low fault tolerance

Another problem is which service should be delegated as the owner, the strategies could be:  
- primary domain priority: domain fitness prioritized  
- operational characteristics priority: performance prioritized  
The primary domain priority is recommended

#### Service Consolidation Technique
Combine multiple table owners (services) into a single consolidated service, thus moving joint ownership into a single ownership scenario; but with more coarse-grained scalability, less fault tolerance, and increased testing scope

### Distributed Transactions

ACID vs BASE: instead of ACID, distributed transactions support BASE.

- BA: Basic Availability
- S: Soft State
- E: Eventual Consistency

### Thinking of distributed transaction
- atomicity: all or nothing
- condition check: transaction conflict; each transaction has a start time & commit time
- order related: order of transactions matters; controls what can be seen, it depends on the isolation level: SI, SSI, etc.

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

## Chapter 10 - Distributed Data Access

### Interservice Communication Pattern

If one service (or system) needs to read data that it cannot access directly, it simply asks the owning service or system for it by using some sort of remote access protocol.

- pros: simplicity, no data volume issues
- cons: network latency (performance) impacts, dependency of other service in critical path, throughput impact coupled, contract required between services

### Column Schema Replication Pattern

Columns are replicated across tables, therefore replicating the data and making it available to other bounded contexts.

- pros: good performance, no service dependency
- cons: data consistency, data synchronization, data ownership

### Replicated Caching Pattern

Replicate data in cache of service node when starting up, and keep updated by communication between the caches or database.

- pros: good performance, data remains consistent, data ownership is preserved
- cons: not good for high data volume or high update rate, initial startup dependency, cloud and containerized configuration can be hard

### Data Domain Pattern

Except for the previous 3 patterns, the only other solution is to create a data domain, combining tables fo the services in the same shared schema, accessible to both services.

- pros: good performance, data remains consistent, no service dependency, no fault tolerance issue, no scalability and throughput issue
- cons: broader bounded context to manage data changes, data ownership governance, data access security

## Chapter 11 - Managing Distributed Workflows

Interaction models in distributed architectures has 3 coupling forces:
- communication: sync / async
- consistency: atomic / eventual
- coordination: choreographed / orchestrated

Orchestration is distinguished by the use of an orchestrator, whereas a choreographed solution does not use one.

### Orchestration Communication Style

A dedicated orchestrator manages the workflow.

Pros:
- Centralized workflow, orchestrator has centralized knowledge
- Error handling, orchestrator owns state for decision assistance
- Recoverability, orchestrator knows what to do
- State management

Cons:
- Responsiveness, potential throughput bottleneck and harm responsiveness
- Fault tolerance, orchestrator is single point of failure, redundancy introduces more complexity
- Scalability, coordination points cut down on potential parallism, so can not scale well
- Service coupling, orchestrator depends on domain components

### Choreography Communication Style

Intent of the communication style that has no central coordination.

Semantic coupling: the inherent coupling exists between domains.

The major lesson of the last decade of architecture design is to model the semantics of the workflow as closely as possible with the implementation.

Pros:
- Responsiveness, less component or network hop
- Scalability, allow independent scaling
- Fault tolerance, can enhance fault tolerance with redundant instances in each component
- Service decoupling, less coupling due to no orchestrator

Cons:
- Distributed workflow, no workflow owner
- State management, no centralized state holder
- Error handling, domain services must have more workflow knowledge to handle errors
- Recoverability, retries and other remediation efforts become more difficult without an orchestrator, domain services need to deal with that

#### Workflow State Management

In orchestration, state is managed by orchestrator; but in choreography, no state management component. There are 3 common options.

- Front controller pattern: the responsibility for state is placed on the first called service in the chain.
	+ pros: creates a pseudo-orchestrator within choreography; makes querying state possible
	+ cons: adds additional workflow state to a domain service; increases communication overhead; harm performance and scale
- Stateless choreography: rely on querying the individual services to build a realtime snapshot.
	+ pros: high performance and scale; extremely decoupled
	+ cons: workflow state must be built on the fly; complexity rises with complex workflows
- Stamp coupling: storing extra workflow state in the message contract sent between services.
	+ pros: allows domains services to pass workflow state without additional queries to a state owner; no front controller
	+ cons: contracts must be larger to accommodate workflow state; doesn't provide just-in-time status query, because no single place to query the state

### Trade-Offs Between Orchestration and Choreography

Trade-offs will lead an architect toward one of these two solutions.

#### State Owner and Coupling

- Choreography suits for workflows need responsiveness and scalability, and either don't have complex error scenarios or they are infrequent
- Orchestration suits for complex workflows that include boundary and error conditions

We need to compare both solutions with the most important requirements, to choose the best suited one.

## Chapter 12 - Transactional Sagas

Coordination is one of the primary forces that create complexity for architects when determining how to best communicate between microservices. In this chapter, we investigate how this force intersects with another primary force, consistency: atomic / eventual

### Transactional Saga Patterns

| Pattern name    | Communication | Consistency | Coordination  |
| --------------- | ------------- | ----------- | ------------- |
| Epic            | Synchronous   | Atomic      | Orchestrated  |
| Phone Tag       | Synchronous   | Atomic      | Choreographed |
| Fairy Tale      | Synchronous   | Eventual    | Orchestrated  |
| Time Travel     | Synchronous   | Eventual    | Choreographed |
| Fantasy Fiction | Asynchronous  | Atomic      | Orchestrated  |
| Horror Story    | Asynchronous  | Atomic      | Choreographed |
| Parallel        | Asynchronous  | Eventual    | Orchestrated  |
| Anthology       | Asynchronous  | Eventual    | Choreographed |

#### Epic Saga(sao) Pattern
- mimic the behavior of monolithic system
- coupling very high due to coordinator
- complexity low due to sync call
- responsiveness/availability low due to sync call and atomic
- scale/elasticity very low due to coordinator

#### Phone Tag Saga(sac) Pattern
- the initial component as front controller, start the chain of communication
- each component must have built-in logic to send compensating requests back along the chain
- this pattern is commonly used for simple workflows that need higher scale, but with a potential performance impact
- coupling high due to sync call and compensating along the chain
- complexity high due to choreography workflow and compensating
- responsiveness/availability low due to sync call and atomic
- scale/elasticity low due to choreography but still tight coupling

#### Fairy Tale Saga(seo) Pattern
- typical fairy tales provide happy stories with easy-to-follow plots
- coupling high due to sync and orchestrator, but without the worse driver of coupling, the atomic transactionality
- complexity low due to convenient options of sync and orchestrator, with loose restriction eventual constitency
- responsiveness/availability medium due to eventual consistency, the mediator can be a simple load balancer (no need to track time-sensitive state within ongoing transaction), but the sync call is not so performant as async call
- scale/elasticity good due to less coupling
- if eventual consistency is acceptable, this pattern is attractive

#### Time Travel Saga(sec) Pattern
- better suited for simple workflow, because lack of coordinator makes it difficult to accommodate complex workflow
- coupling medium due to only sync call, without orchestrator and atomicity
- complexity low due to no transaction, suited to fast throughput, one-way communication architectures
- responsiveness/availability medium due to good performance, but low for complex error handling, because in lack of orchestrator
- scale/elasticity good due to less coupling
- sync communication is easier than async, if this pattern can provide adequate scalability, teams don't have to embrace the complex but more scalable alternative of Anthology Saga(aec)

#### Fantasy Fiction Saga(aao) Pattern
- similar to Epic Saga(sao), with async communication, atomic requires result, but async has to wait for result, it is not a common usage
- coupling high due to atomicity and orchestrator
- complexity high due to high coupling and complex communication, as well as the debug and ops effort
- responsiveness/availability low due to transactional requirement
- scale/elasticity low due to transactional requirement
- this pattern is unfortunately more popular than it should be, mostly from the misguided attempt to improve the performance of Epic Saga(sao) while maintaining transactionality; a better option is usually Parallel Saga(aeo)

#### Horror Story(aac) Pattern
- this pattern is the worst possible combination
- coupling medium due to only atomicity coupling requirement
- complexity extremely high due to the most stringent requirement (transactionality) with the most difficult combination of other factors to achieve that (async and choreography)
- responsiveness/availability low due to transactional requirement
- scale/elasticity medium due to async and choreography

#### Parallel Saga(aeo) Pattern
- the Parallel Saga(aeo) pattern is named after the “traditional” Epic Saga(sao) pattern with two key differences that ease restrictions and therefore make it an easier pattern to implement: asynchronous communication and eventual consistency
- coupling low due to loose requirement of async and eventual consistency
- complexity low due to loose coupling and simple requiement of workflow in mediator
- responsiveness/availability high due to async call and non-transactional requirement
- scale/elasticity high due to async communication and non-transactional requirement, each component can scale by demand
- overall, the Parallel Saga(aeo) pattern offers an attractive set of trade-offs for many scenarios, especially with complex workflows that need high scale

#### Anthology Saga(aec) Pattern
- this pattern works best for simple, mostly linear workflows, where architects desire high processing throughput
- coupling very low due to the loosest coupling requirement
- complexity high due to no orchestrator, thus each component handles its own communication and errors
- responsiveness/availability high due to async call and non-transactional requirement
- scale/elasticity very high due to loose coupling
- the Anthology Saga(aec) pattern is well suited to extremely high throughput communication with simple or infrequent error conditions

### State Management and Eventual Consistency
- state managed in state machine makes it good responsiveness and less impact to end users
- however, the disadvantage is data may be out of sync when error occurs, and eventual consistency takes time

### Techniques for Managing Sagas
annotation in java or attribute in c# can help indicating the transaction workflows involved services, so architects and developers can know the component change might impact which workflow and which other components, thus the saga is stored in the code base

## Chapter 13 - Contracts

### Strict Versus Loose Contracts
- strict contracts: XML Schema, JSON schema, Object, RPC (including gRPC)
- neutral contracts: GraphQL, REST
- loose contracts: value-driven contract, simple JSON, KVP arrays (maps)

#### Trade-Offs Between Strict and Loose Contracts

##### Strict contracts
- pros: guaranteed contract fidelity, version for evolution, verify at build time, better documentation
- cons: tight coupling, version brings more effort and precautions

##### Loose contracts
- pros: highly decoupled, easier to evolve
- cons: contract management costs effort, requires fitness functions in consumer side

#### Contracts in Microservices
- Coupling levels: contract between microservices is better to be highly decoupled, so loose contract preferred; but it brings the problem of contract fidelity
- Consumer-driven contracts: consumer creates a contract specified information and passes it to the provider, the provider includes the test of consumer required information as part of continuous integration (CI) of development pipeline.
	+ it is common in microservice architecture because it allows loose coupling and governed integration
	+ pros: loose coupling, variability in strictness, evolvable
	+ cons: require engineering maturity, two interlocking mechanisms rather than one

### Stamp Coupling
Each service accesses only a small portion of the data structure passed between each service. For example, the travel industry has a global standard XML document format that specifies details about travel itineraries.

- Over-Coupling via Stamp Coupling: too much unused information in contract
- Bandwidth: costs more bandwidth

#### Stamp Coupling for Workflow Management
For complex workflow, choreography is difficult, but stamp coupling can be used to handle it, passing both domain knowledge and workflow state as part of the contract.

Trade-offs of stamp coupling:
- pros: allows complex workflow within choreographed solutions
- cons: creates high coupling between collaborators; can create bandwidth issues at high scale

## Chapter 14 - Managing Analytical Data

### Data Warehouse
- Data extracted from many sources
- Transformed to single schema
- Loaded into warehouse
- Analysis done on the warehouse
- Used by data analysts
- BI reports and dashboards
- SQL-ish interface
- Start schema pattern was popular to facilitate simpler queries

Problems:
- Integration brittleness
- Extreme partitioning of domain knowledge
- Complexity
- Limited functionality for intended purpose
- Synchronization creates bottlenecks
- Operational versus analytical contract differences

### Data Lake
It inverts the “transform and load” model of the data warehouse to a “load and transform” one, rather than do work that might not be needed, do transformation work only on demand.
- Data extracted from many sources
- Loaded into the lake
- Used by data scientists

Problems:
- Difficulty in discovery of proper assets
- PII and other sensitive data
- Still technically, not domain, partitioned

### Data Mesh

#### Definition of Data Mesh
- Domain ownership of data
- Data as a product
- Self-serve data platform
- Computational federated governance

#### Data Product Quantum
Just as in the service mesh, teams build a data product quantum (DPQ) adjacent but coupled to their service. 

Several types of DPQs commonly exist in modern architectures:
- Source-aligned (native) DPQ
- Aggregate DQP
- Fit-for-purpose DPQ

Each data product quantum acts as a cooperative quantum for the service itself: Cooperative quantum. They are tightly coupled via async communication and eventual consistency, but looser contract coupling to the analytics quantum.

#### Data Mesh, Coupling, and Architecture Quantum
- Data quantum should be orthogonal to service quantum, so they should be async communicated, and eventual consistency
- Parallel Saga(aeo) or Anthology Saga(aec) can be used between them

#### When to Use Data Mesh
- Highly suitable for microservices architectures
- Follows modern architecture principles and engineering practices
- Allows excellent decoupling between analytical and operational data
- Carefully formed contracts allow loosely coupled evolution of analytical capabilities
- Requires contract coordination with data product quantum
- Requires asynchronous communication and eventual
consistency

## Chapter 15 - Build Your Own Trade-Off Analysis

Three-step process for modern trade-off analysis
- Find what parts are entangled together
- Analyze how they are coupled to one another
- Assess trade-offs by determining the impact of change to interdependent systems

### Finding Entangled Dimensions
- static coupling diagram
	+ OS/container dependency
	+ transitive dependency (via lib, framework, etc.)
	+ persistence dependency on DBs, search engine, cloud env, etc.
	+ arch integration points required for service bootstrap
	+ messaging infra required for communication
- analyze coupling points
	+ goal of analysis is to determine what forces the architect needs to study
	+ list all the Saga patterns, analyze which dimensions are important
	+ the more coupling present in the pattern, the worse its scalability
- assess trade-offs
	+ determine the must dimension, then compare to choose the rest ones

### Trade-Off Techniques
- Qualitative Versus Quantative Analysis
	+ qualitative analsis is enough to compare two architectures, they will always differ enough to prevent true quantitative comparisons
- MECE Lists: Mutually exclusive & Collectively exhaustive
- The “Out-of-Context” Trap
	+ common solution always not work without the specific context
- Model Relevant Domain Cases
	+ common integrators and disintegrators can assist compare two options, but focused scenarios or use cases help to make the decision
	+ As architecture generally evades generic solutions, it is important for architects to build their skills in modeling relevant domain scenarios to home in on better trade-off analysis and decisions
- Prefer Bottom Line over Overwhelming Evidence
	+ for non-tech stakeholders, they don't need overwhelming tech details
	+ a bottom line comparison can help them make the decision of trade-off between biz and tech
- Avoiding Snake Oil and Evangelism
	+ there's no common solution, everything has trade-offs
	+ architects should avoid evangelizing and try to become the objective arbiter of trade-offs


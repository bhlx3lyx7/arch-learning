# Architecting Distributed Transactional Applications

## Chapter 1 - Planning for a Distributed Transactional Application

### The Business Drivers for Distributed Systems

- Reliability
- Scalability: scale out
- Elasticity: scale down
- Performance
	+ higher throughput by multi nodes
	+ lower latency by locality

### The Increasingly Attractive Economics of Distributed Computing

Revolutionary technologies over the past 10-15 years:
- Public cloud computing
- Containerization technologies: Docker
- Container orchestration solutions: Kubernetes

### Understanding Your Requirements

- Total cost of ownership
	+ hardware cost for on-premises deployment
	+ hardware rental cost for cloude deployment
	+ software licensing cost
	+ staffing cost for administrators
- High availability requirements: trade-off with cost
- Throughput and latency requirements: trade-off between them
- Geographical considerations: trade-off with privacy and data domiciling regulations in different regions

### A Modern Distributed and Transactional Architectural Pattern

- public cloud platform as primary substrate
- microservice pattern for top-level application architecture
- docker container as packaging for microservices
- kubernetes to orchestrate deployment and maintenance
- optionally deploy a message brokering layer like kafka
- database platform for persistence, which is fully managed, cloud-based, distributed, transactionally consistent

## Chapter 2 - Distributing the Application Layer

### Regions and Zones

- region, include multiple zones, always a specific country or city
- zone, usually a specific data center, typically at least three zones per region

### Microservices

- A microservice should have a single concern
- The microservice should be independent of all other services
- The microservice should be small enough to be developed by a single team
- The microservice should be ephemeral
- The microservice implementation should be opaque

### Kubernetes, Pods, and Services

- kubernetes provides application service, exposed logically
- pod is the manage unit of virtualized resources, a service is supported multiple pods, across different nodes or zones
- container is included in a pod, a pod could hold multiple containers, sharing the same resources; always an app container with some sidecar containers

### Multiregion Kubernetes

- while it is techinally possible to deploy a stretched kubernetes cluster with nodes across multiple regions, the latency penalty that results is likely to be intolerable
- thus kubernetes clusters are generally confined to a single region, and can span multiple zones, for higher availability
- a global load balancer is used to route requests to appropriate cluster
- between different cloud platforms, third-party global load balancing services are offered by major CDN vendors

### Event Management

- distributed messaging service is used to store short-term persistent events, to support asynchronous communication between microservices
- transient messages between services always do not pass between regions, due to consideration of latency and security
- to tolerate failure of a region, messages can be replicated to another region asynchronously, some messages might be lost, but hopefully the bulk of work can be successfully transferred to the new region in such a disaster

### Serverless Deployments

- serverless platforms can provide a simpler solution for deploying individual services
- serverless platforms offer increased simplicity at the expense of reduced flexibility
- serverless platforms can place restrictions on container sizes or programming languages that are supported
- security policies might require that application code not run on shared cloud instances, which might rule out a serverless deployment

With a serverless deployment, developers can
1. implement a service
2. invoke serverless platform command to deploy
3. scale and manage the service via platform's management interface

### Multiregion Serverless

serverless deploymentss are usually enabled in specific regions only, so multiregion serverless solution will need a global load balancer for traffic routing

## Chapter 3 - Distributing and Scaling the Storage Layer

### Transactional Versus Nontransactional Distributed Databases

CAP theroem: you can only have at most two of three desirable characteristics in a distributed system
- Consistency: very user sees the same view of the database state
- Availability: The database remains available unless all elements of the dis‐ tributed system fail
- Partition tolerance: The system can continue to operate in the event that a network partition divides the distributed system in two or if two nodes in the network cannot communicate

Public cloud platform with redundant links between regions can reduce the possibility of network partitions, thus consistency and availability can be achieved together.

### Hosting Strategies for Distributed Databases

#### Do-it-yourself on your own hardware

pros:
- Maximum control over hardware and OS configuration
- Lower ongoing hardware "rental" costs compared to a cloud deployment
- Reduced latency for applications running on premises

cons:
- The highest cost in terms of skilled human resources
- High initial cost in terms of hardware acquisition
- Paying for computing resources that are unused during idle periods

#### Do-it-yourself on a cloud platform

pros:
- Ability to reconfigure computing resources dynamically
- Reduced capital hardware expenditure
- Availability of additional services such as Amazon S3 for backup storage
- Ability to fine-tune placementof nodes to minimize latency

cons:
- Increased operational expenditure (hardware "rental")

#### Fully managed cloud database service

pros:
- Reduced operational costs
- Rapid deployment
- Rapid scaling and reconfiguration

cons:
- Some loss of fine-grained control over software and hardware configuration

#### Serverless cloud database

pros:
- Minimal operational costs
- Pay only for the resources you use
- No need for predeployment capacity planning
- Automatic and seamless scaling with demand

cons:
- Your deployment is cotenanted with other serverless users. This may result in less predictable performance when compared to a dedicated deployment. 
- This approach may conflict with some security policies regarding colocation of data with other tenants.
- Billingmaybeunpredictableif monthly budgets are set too high, or performance may be throttled if monthly budgets are set too low

Two major reasons to choose applications and databases "all cloud" or "all on premises":
- Security: cross communication would increase vulnerability to a cyberattack
- Latency: locality access would have lower latency

### Serverless or Dedicated Deployment?

Advantages of serverless:
- pay only for resources you use
- resources scales dynamically, no need to determine ahead of time
- develop with low or free resource, and seamlessly transition to a paid service when move into production

Limitations of serverless:
- physical resources are shared, which might be unacceptable due to security considerations
- cotenanting could lead to noisy neighbor, which disrupt your performance
- cost and resource utilization is not relatively fixed as in dedicated mode, so billing or performance might be unpredictable
- currently available serverless database offering are sometimes tied to a specific region, which may cause performance issues if the application is distributed globally
- not all database vendors offer a serverless deployment option

### Kubernetes

The workloads type of database is very different from in application layer, so database layer and application layer would be better NOT to locate in the same type of Kubernetes cluster.

### Placement Policies

Fully managed dedployment reduces administrative costs and operational risks. 
However, the placement of nodes is usually not exposed to users. e.g. cloud platform can encourage nodes to be located physically close to each other, such as in the same rack; but it is not available when using a fully managed service.

### Multiregion Database Deployments

In almost all cases, database cluster is deployed similarly to its related application cluster, to make the communication regions and zones are NOT mismatched, for lower latency.

### Distributed Database Consensus

Distributed consensus protocol allows all the nodes that comprise the database to agree on the current state of any data item

- replication factor: number of copies
- majority of replicas: to tolerate partial failure
- trade-off between performance and availability: local writes have better performance, but distributing replicas can tolerate region failure

### Survival Goals

- Zone failure survival goal: tolerate a node or zone failure
- Region failure survival goal: tolerate a region failure

zone survival results in the best performance, while regional survival promotes greater survivability in case of large-scale outage or network partitions.

### Locality Rules

- Global table: whole table in any region
	+ write slow, read fast
	+ e.g. static information like product table
- Regional table: whole table within one region
	+ local write/read fast, remote write/read slow
	+ e.g. information within local site
- Regional-by-row table: rows are distributed in specific regions
	+ local write/read fast, remote write/read slow
	+ e.g. user records can be located in the nearest region of their own countries

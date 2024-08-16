# Apache Ignite

Apache Ignite is a memory-centric distributed database, caching, and processing platform for transactional, analytical, and streaming workloads, delivering in-memory speeds at petabyte scale.

## Usage
typical usage
- memory-centric distributed database
- caching
- processing

can also be used as
- complete in-memory database: if disk persistence disable, it can also work within memory
- multi-model database: key-value model by default, can provide SQL to access data in other models
- transactional database: support transactions at key-value level; can span in different partitions on different servers
- microservice platform: ability to store and process at the same resources, to build fault-tolerant, scalable microservices
- in-memory data fabrics: delivers functionalities such as 
	+ big data accelerator
	+ web session clustering
	+ spring cache
	+ implementation for cluster manager

### Suit scenarios
1. High-volume of ACID transactions processing. 2. Cache as a Service (CaaS).
3. Database Caching.
4. On-line fraud detection.
5. Complex event processing for IoT projects.
6. Real-time analytics.
7. HTAP business applications.
8. Fast data processing or implementing lambda architecture.

## Compare with Redis
Apache Ignite and Redis are both in-memory data storage systems that offer high-performance and scalability. Let's explore the key differences between them.
- Data Model: Apache Ignite offers a flexible data model that supports key-value, SQL, and compute grid functionalities. It allows users to store and manipulate complex structured data using SQL queries, distributed compute operations, and in-memory key-value stores. On the other hand, Redis primarily focuses on a simple key-value data model, offering basic data structures like strings, lists, sets, and hashes.
- Durability: Apache Ignite provides durability by automatically storing data in memory as well as on disk, ensuring data persistence even after system restarts or failures. It supports write-ahead logging, replication, and data partitioning across the cluster. While Redis also supports data persistence, it typically relies on periodic snapshots and append-only log files for durability, which may result in some data loss in case of system failures.
- Scalability: Apache Ignite offers horizontal scalability by distributing data across the cluster nodes using data partitioning techniques. It can handle much larger datasets and higher workloads by leveraging distributed computing capabilities. Redis, on the other hand, has a single-threaded architecture that can limit its scalability for certain use cases, although it offers a clustering feature to scale out by sharding data across multiple nodes.
- Supported Data Types: Redis provides a rich set of built-in data structures like strings, lists, sets, sorted sets, and hashes, allowing users to perform various operations on these data types. Apache Ignite, in addition to key-value operations, supports more complex data structures like collections, maps, and SQL tables, enabling users to perform advanced computations and queries on the stored data.
- Persistence Options: Redis offers different persistence options, including both snapshotting and append-only log files (AOF). Users can choose between these options based on their requirements for data durability and recovery. Apache Ignite, on the other hand, supports write-ahead logging to ensure data durability and provides multiple choices for storage, including in-memory, disk-based, or a combination of both, allowing users to optimize performance and persistence based on their specific needs.
- Parallel Query Processing: Apache Ignite supports distributed SQL queries, allowing users to execute complex queries across the entire dataset in a parallel and distributed manner. It leverages its distributed computing capabilities to optimize query processing and achieve faster query response times. Redis, on the other hand, does not provide built-in support for distributed SQL queries and primarily focuses on key-value operations.

In summary, Apache Ignite is a distributed in-memory computing platform that integrates with existing data sources and provides features like distributed caching, compute grid, and streaming processing, making it suitable for large-scale, high-performance data processing and analytics. Redis, on the other hand, is an open-source, in-memory data structure store known for its simplicity, speed, and versatility, primarily used for caching, real-time analytics, and message brokering in web applications.

## Reference
- official site: https://ignite.apache.org/
- book: The Apache Ignite Book
- compare with redis
	+ https://redisson.org/feature-comparison-redis-vs-ignite.html
	+ https://stackshare.io/stackups/apache-ignite-vs-redis
	+ https://db-engines.com/en/system/Hazelcast%3BIgnite%3BRedis
---
title: "Scalability"
sort_date: 2025-08-24
---

## Vertical Scaling vs Horizontal Scaling

**Vertical scaling**, referred to as "scale up", means the process of adding more power (CPU, RAM, etc.) to your servers.

- A great option if traffic is low
- However, it is impossible to add unlimited CPU and memory to a single server
- Vertical scaling does not have failover and redundancy. If one server goes down, the website/app goes down with it completely.

**Horizontal scaling**, referred to as "scale out", allows you to scale by adding more servers into your pool of resources.

- More desirable for large scale applications


## Load Balancing
**Load balancing** is the process of distributing incoming network traffic or workload across multiple servers, systems, or resources to ensure no single server gets overwhelmed.

A **load balancer** sits between clients and servers. When a client makes a request, the load balancer decides which server should handle it using a chosen strategy.

**Common strategies**
- **Round robin:** Requests are sent to each server in turn.
- **Least connections:** Requests go to the server handling the fewest active connections.
- **IP hash:** The client’s IP determines which server they connect to.
- **Weighted distribution:** Servers with more capacity get more requests.


## Database Replication
Database replication is the process of copying and maintaining data across multiple databases or servers to improve availability, reliability, and performance.

### Purpose of Replication

- **High availability:** If one database server goes down, another replica can take over.
- **Load balancing:** Read queries can be distributed across replicas, reducing pressure on the primary server.
- **Disaster recovery (Reliability):** Replicas can serve as backups in case of data loss or corruption.
- **Geographic distribution:** Data can be replicated to servers closer to users in different regions to reduce **latency**.

### Types of Replication

#### Master-Slave (Primary-Replica)
- One primary database handles writes. Replicas get copies of the data and handle read operations.
- Most applications require a much higher ratio of reads to writes; thus, the number of slave databases in a system is usually larger than the number of master databases.

#### Master-Master (Multi-Primary)
- Multiple databases can handle both reads and writes. Requires conflict resolution if two primaries update the same record.

#### Synchronous vs. Asynchronous
- Synchronous: The primary waits until data is written to replicas before confirming a transaction (safer, but slower).
- Asynchronous: The primary confirms writes immediately, and replicas catch up later (faster, but risks lag or data loss on failure).


## Cache
A cache is a temporary storage area that stores the result of expensive responses or frequently accessed data in memory so that subsequent requests are served more quickly. A cache is a simple **key-value store** and it should reside as a buffering layer between your application and your data storage. Whenever your application has to read data it should at first try to retrieve the data from your cache. Only if it’s not in the cache should it then try to get the data from the main data source.

### Two Patterns of Caching

#### Cached Database Queries
- You store the results of database queries in the cache, using a hash of the query as the cache key
- Drawbacks: This pattern can become messy when data changes. If one piece of data (e.g., a table cell) updates, you must identify and delete all cached query results that might include that data

#### Cached Objects
- Instead of caching query outputs, cache assembled objects — such as complete instances of a class
- How it works: Suppose you have a `Product` class with data including prices, descriptions, pictures, and reviews aggregated via multiple methods. Once you've assembled this dataset, store the object (or the data array) directly in the cache
- Benefits:
    - Easier cache invalidation when an object updates
    - More logical and maintainable code architecture
    - Enables asynchronous processing: you could have workers preassemble objects in the cache, letting the application fetch the latest version without needing to hit the database during user requests


### Considerations for using cache
- **Decide when to use cache:** consider using cache when data is read frequently but modified infrequently.
- **Expiration policy:** it is advisable not to make the expiration date too short as this will cause the system to reload data from database too frequently. Meanwhile, it is advisable not to make the expiration date too long as the data can become stale.
- **Consistency:** keep the data store and the cache in sync.
- **Mitigating failures:** multiple cache servers across different data centers are recommended to avoid SPOF (single point of failure)
- **Eviction policy:** once the cache is full, any requests to add items to the cache might cause existing items to be removed. This is called cache eviction. **Least-recently-used (LRU)** is the most popular cache eviction policy.
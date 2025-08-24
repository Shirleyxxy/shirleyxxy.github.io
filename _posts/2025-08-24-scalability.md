---
title: "Scalability"
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
---
title: "Big Data Ecosystem"
---

## What is Hive?

Apache Hive is a **data warehouse system** built on top of Hadoop.

It provides an SQL-like interface (HiveQL) to query data stored in HDFS (Hadoop Distributed File System) or other compatible storage systems.

Hive itself is not a database — it doesn’t store the data; rather, it stores metadata about the data (schemas, tables, partitions) in a Metastore (usually backed by MySQL/Postgres).

When you run a query in Hive, it translates your SQL-like query into MapReduce/Spark/Tez jobs under the hood.

In short: Hive = **SQL on top of Hadoop/Spark for querying big data**.
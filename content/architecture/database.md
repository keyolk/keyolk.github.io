+++
title  = "database"
toc    = true
weight = 1
+++

## Relational Database
#### ACID
- Atomicity
- Consistency
- Isolation
- Durability

## NoSQL
### CAP
- Consistency
- Availability
- Partition tolerance

![img](../database.files/nosql_cap.png)

각각 조합에 대한 예: \

- AP: Dynamo, Cassandra, SimpleDB, CouchDB
- CA: Aster Data, Greenplum
- CP: BigTable, Hbase, MongoDB, Redis, MemcacheDB

### BASE
- Basically Available
- Soft-state
- Eventually consistency

### PACELC
> if there is a partition (P) how does the system tradeoff between availability and consistency (A and C); else (E) when the system is running as normal in the absence of partitions, how does the system tradeoff between latency (L) and consistency (C)?

- Hbase : PC/EC
- Cassandra : PA/EL

![img](../database.files/nosql_pacelc.png)

## Data Model
- relational
- key-value
- column-oriented/tabular
- document-oriented

## Data Type
- scalar: number, string, etc
- multi-valued: sets

{{%attachments title="Related files" pattern=".*"/%}}

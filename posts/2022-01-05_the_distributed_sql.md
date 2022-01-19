---
title: "The Distributed SQL Database: a database evolution or a domain-driven optimization?"
date: 2022-01-05T10:21:09+08:00
draft: false
tags: [技术,en]
categories: [闲白]
canonicalUrl: https://jungu.me/post/the-distributed-sql/
---

Different companies and venture capitals have invested billions of bucks in this technology.

|  Company  | Funding Round | Total Funding Amount | Valuation  |
| :-------: | :-----------: | :------------------: | :--------: |
| Cockroach |   Series F    |      USD $633M       |  USD $5B   |
|  PingCAP  |   Series D    |     USD $341.6M      |  USD $3B   |
| Yugabyte  |   Series C    |      USD $291M       | ～ USD $1B |

Most high-profile distributed SQL databases use the same method: redo-log replication by consensus algorithms, e.g., Raft consensus algorithm. This is synchronous processing using consensus algorithms to build a distributed shared memory for redo-log records.

In my opinion, redo-log replication by consensus algorithms has been the most overestimated database technology in recent years. There are two key issues.

## The fragile presumption of redo-log replication is better

Let's say we have two database nodes, the source node and the target node. In the ever beginning, they have the exact copy of the data. We have some change operations (I/U/D SQL statements) on the source node, and we want to keep the target node updated. What should we do? The simplest way is to replay the operations on the target node. But this is not the most efficient way.

Thinking about the running cost of an I/U/D statement, we can divide it into the execution preparation and the physical work parts. The execution preparation part includes the work of SQL parser, SQL optimizer, etc. No matter how many data records will be affected, it is a fixed cost. The cost of the physical work part depends on how many data records will be affected; it is a floating cost. The idea behind redo-log replication is to save the fixed cost on the target node; we only replay the redo-log (the physical work) on the target node.

The cost-saving percentage is the reciprocal of the number of redo-log records. If one operation only affects one record, we should see significant savings from redo-log replication. What if it's 10,000 records? Then we should worry about the network reliability. Which one is more reliable, send the one operation or the 10,000 redo-log records? How about one million records? Redo-log replication is super in scenarios like payment systems, metadata systems, etc. In these scenarios, each database I/U/D operation only affects a small number of records (1 or 2). But it's hard to work with I/O intensive workloads like batch jobs in a banking ledger system or a trading clearing system.

## Vendors mislead people's expectations on the capability of consensus algorithms

Vendors always claim consensus algorithms could provide strong consistency to the database clusters. But people only use consensus algorithms to replicate the redo-log records. The redo-log records are consistent on different nodes, but that doesn't mean the data views on other nodes are consistent either. We have to merge the redo-log records into the actual table records. So even with this synchronous processing, we can still only get eventual consistency on the data views.

## Something different: cloud-native databases

- [Socrates: The New SQL Server in the Cloud](https://www.microsoft.com/en-us/research/uploads/prod/2019/05/socrates.pdf)
- [Milvus cloud-scalable vector database](https://milvus.io/docs/v2.0.0/overview.md)

---
title: Size your infrastructure
linkTitle: Size your infrastructure
weight: 20
date: 2022-06-09
description: Size your infrastructure to be able to store and process millions of transactions per day and make them quickly available for traffic monitoring and analytics.
---

Operational Insights is designed to store and process millions of transactions per day and make them quickly available for traffic monitoring and analytics. This advantage of being able to access millions of transactions is not free of charge with Elasticsearch, but is available in the size of the disc space provided. The solution has been extensively tested, especially for high-volume requirements. It processed 1010 transactions per second, up to 55 million transactions per day on the following infrastructure.

## Sizing recommendations

The following are important aspects for sizing your platform:

* Transactions per second - transactions to be processed in real time.
* Retention period - this is reflected in the required disk space.

### Transactions per second

The number of concurrent transactions per second (TPS) that the entire platform must handle. The platform must therefore be scaled so that the events that occur on the basis of the transactions can be processed (Ingested) in real time. It is important to consider the permanent load. As a general rule, more capacity should be planned in order to also quickly enable catch-up operation after a downtime or maintenance.

(to be continued)

### Retention period

The second important aspect for sizing is the retention period, which defines how long data should be available. Accordingly, disk space must be made available.
In particular the Traffic-Summary and Traffic-Details indicies become huge and therefore play a particularly important role here. The solution is delivered with default values which you can read here. Based on the these default values which result in ap. 14 days the following disk space is required.

(to be continued)

### Results of infrastructure test

(to be continued)
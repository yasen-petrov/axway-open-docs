---
title: Configure API Gateway Manager for Elasticsearch
linkTitle: Configure API Gateway Manager
weight: 30
date: 2022-06-09
description: Configure API Gateway Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances.
---

As the idea of this project is to use the existing API-Gateway Manager UI (short: ANM) to render log data now provided by Elasticsearch instead of the individual API-Gateway instances before (the build in behavior), it is required to change the ANM configuration to make use of Elasticsearch instead of the API-Gateway instances (default setup). By default, ANM is listening on port 8090 for administrative traffic. This API is responsible to serve the Traffic-Monitor and needs to be configured to use the API-Builder REST-API instead.

## S

The following are important aspects for sizing your platform:

* Transactions per second - transactions to be processed in real time.
* Retention period - this is reflected in the required disk space.

### a

The number of concurrent transactions per second (TPS) that the entire platform must handle. The platform must therefore be scaled so that the events that occur on the basis of the transactions can be processed (Ingested) in real time. It is important to consider the permanent load. As a general rule, more capacity should be planned in order to also quickly enable catch-up operation after a downtime or maintenance.

(to be continued)

### Results of infrastructure test

(to be continued)

## Next steps

On the following section, you are going to configure your API Gateway Manager to render log data, which is now provided by Elasticsearch instead of the individual API Gateway instances. For more information, see [Configure API Gateway Manager for Elasticsearch](/amplify_analytics/config_APIMng_elasticsearch).
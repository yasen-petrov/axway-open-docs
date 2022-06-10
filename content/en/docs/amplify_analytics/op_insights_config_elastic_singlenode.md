---
title: Configure a single node Elasticsearch cluster
linkTitle: Basic configuration
weight: 30
date: 2022-06-09
description: Basic configuration to test a single node Elasticsearch cluster, including a Kibana instance running.
---

The basic setup explains the individual components, how they can be deployed and play together. After completing the basic setup you will have a single node Elasticsearch cluster including a Kibana instance running. This cluster receives data from 1 to N API-Gateways via Filebeat, Logstash, API-Builder and is accessible via the Traffic Monitor. You can also import and use the sample Kibana Dashboard or create your own visualizations.

However, the basic setup uses minimal parameters to run and test the solution on a machine with at least 16 GB of RAM including the API-Management platform (For instance like the Axway internal API-Management reference environment.).
Therefore, the solution is not suitable for a production environment without further configuration. For a production environment, check the parameters mentioned in the env-sample at the beginning and set them if necessary.

placeholder

## Next steps

On the following section, you are going to ...
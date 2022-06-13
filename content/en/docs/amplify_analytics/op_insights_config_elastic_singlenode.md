---
title: Configure a single node Elasticsearch cluster
linkTitle: Basic configuration
weight: 20
date: 2022-06-09
description: Basic configuration to test a single node Elasticsearch cluster with multiple Kibana dashboards.
---

The basic setup explains the individual components, how they can be deployed and work together.

{{< alert title="Note">}}This configuration is only recommended for development environments.{{< /alert >}}

After completing the basic setup you will have a single node Elasticsearch cluster including multiple Kibana dashboards. At this point, API Gateway Manager is not yet configured, therefore this single cluster receives data from 1 to many API Gateways via Filebeat, Logstash, and API Builder, and is accessible via the Traffic Monitor.

You can also import and use the sample Kibana Dashboard or create your own visualizations.

This basic setup uses minimal parameters to run and test the solution on a machine with at least 16 GB of RAM including the API Management platform (for example, like the Axway internal API-Management reference environment).
Therefore, the solution is not suitable for a production environment without further configuration. For a production environment, check the parameters mentioned in the env-sample at the beginning and set them if necessary.

placeholder

## Next steps

After you have tried the solution in a single node scenario, the next steps is to [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) before configuring Operational Insights in your production environment.
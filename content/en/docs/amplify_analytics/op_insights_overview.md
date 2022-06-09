---
title: Overview of Operational Insights
linkTitle: Overview
weight: 10
date: 2022-06-09
description: Overview of Amplify Analytics Operational Insights and high-level steps required to set up and use it.
---

Operational Insights is part of the Amplify Analytics solution(/docs/apimanager_analytics/analytics_intro/). It works on top of the analytics stack, [Elasticsearch, Logstash, and Kibana](https://www.elastic.co/elasticsearch/) (ELK), to allow you to observe millions of requests across different API Gateway instances in a long time frame.

Operational Insights uses Logstash as the data source (instead of OPSDB) for API Gateway built-in [Traffic Monitor](/docs/apimanager_analytics/analytics_intro/), which resolves the performance and scalability challenges with OPSDB.

![API Gateway Analytics](/Images/concepts/reporter.png)

API Management Operational Insights components provides the following key benefits:

**Performance**:

When having many API Gateway instances with millions of requests, the API Gateway traffic monitor (link to classic monitor in our docs) can become slow and the observation period quite short. Operational insights solve that performance issue, and make it possible to observe a long time-frame and get other benefits by using a standard external datastore, the Elasticsearch

**Visibility**:

The solution allows API Manager users to use the standard traffic monitor and see only the traffic of their own APIs. This allows API service providers who have registered their APIs to monitor and troubleshoot their own traffic without the need for a central team.

**Analytics**:

Deliver standard dashboards that provide analysis capabilities across multiple perspectives. It also allows you to add your own dashboards.

## Prerequisites

Deploying this component requires knowledge of the Axway API Management solution, basic understanding of Docker, and a solid understanding of [HTTPS and server certificates and how to validate them via trusted CAs](https://www.ssl.com/article/browsers-and-certificate-validation/).

### Docker

Components such as the API-Builder project are supposed to run as a Docker-Container. The Elasticsearch stack is using standard Docker-Images which are configured with environment variables and some mount points. With that, you are pretty flexible. You can run them with the provided docker-compose or with a Docker Orchestration platform such a Kubernetes or OpenShift to get elastic scaling & self-healing.

### Docker-Compose or HELM

Docker Compose is one option to deploy the solution. Additionally a [Helm chart](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/helm/README.md) is provided to deploy the solution on Kubernetes or OpenShift.

### API Gateway and APIManagement

The solution is designed to work with Classical and the EMT API Management deployment model. As it is mainly based on events given in the Open-Traffic-Event log, these must be enabled. Also Events-Logs are indexed and stored in Elasticsearch. This is used for System-Monitoring information and to highlight annotations based on Governance-Alerts in API-Manager.
Version 7.7-20200130 is required due to some Dateformat changes in the Open-Traffic-Format. With older versions of the API-Gateway you will get errors in the Logstash processing.

### Elastic stack

The solution is based on the Elastic-Stack (Elasticsearch, Logstash, Beats and Kibana). It can run completely in docker containers, which for example are started on the basis of docker-compose.yaml or run in a Docker Orchestration Framework.
It is also possible to use existing components such as an Elasticsearch cluster or a Kibana instance. With that you have the flexiblity to use for instance an Elasticsearch service at AWS or Azure or use Filebeat manually installed on the API-Gateway machines. The solution has been tested with Elasticsearch >7.10.x version.

For more details on managing metrics, see [Purge the metrics database](/docs/apimanager_analytics/metrics_db_purge).

## Monitoring and reporting with API Gateway Analytics

When you have configured the metrics database and API Gateway Analytics, you can start monitoring your API traffic and generating reports in API Gateway Analytics. For details, see [Get started with API Gateway Analytics] (/docs/apimanager_analytics/analytics_start/).

## Next steps

The next important aspect to consider before configuring Operational Insights is to size your Size your infrastructure. For more information, see [Infrastructure sizing recommendations](/amplify_analytics/infra_size).

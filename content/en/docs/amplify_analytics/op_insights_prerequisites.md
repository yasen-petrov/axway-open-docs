---
title: Operational Insights prerequisites
linkTitle: Prerequisites
weight: 10
date: 2022-06-09
description: Prerequisites to configure Operational Insights in API Gateway.
---

Deploying this component requires knowledge of the Axway API Management solution, basic understanding of Docker, and a solid understanding of [HTTPS and server certificates and how to validate them via trusted CAs](https://www.ssl.com/article/browsers-and-certificate-validation/).

## Docker

Components such as the API-Builder project are supposed to run as a Docker-Container. The Elasticsearch stack is using standard Docker-Images which are configured with environment variables and some mount points. With that, you are pretty flexible. You can run them with the provided docker-compose or with a Docker Orchestration platform such a Kubernetes or OpenShift to get elastic scaling & self-healing.

## Docker-Compose or HELM

Docker Compose is one option to deploy the solution. Additionally a [Helm chart](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/helm/README.md) is provided to deploy the solution on Kubernetes or OpenShift.

## API Gateway and APIManagement

The solution is designed to work with Classical and the EMT API Management deployment model. As it is mainly based on events given in the Open-Traffic-Event log, these must be enabled. Also Events-Logs are indexed and stored in Elasticsearch. This is used for System-Monitoring information and to highlight annotations based on Governance-Alerts in API-Manager.
Version 7.7-20200130 is required due to some Dateformat changes in the Open-Traffic-Format. With older versions of the API-Gateway you will get errors in the Logstash processing.

## Elastic stack

The solution is based on the Elastic-Stack (Elasticsearch, Logstash, Beats and Kibana). It can run completely in docker containers, which for example are started on the basis of docker-compose.yaml or run in a Docker Orchestration Framework.
It is also possible to use existing components such as an Elasticsearch cluster or a Kibana instance. With that you have the flexiblity to use for instance an Elasticsearch service at AWS or Azure or use Filebeat manually installed on the API-Gateway machines. The solution has been tested with Elasticsearch >7.10.x version.

For more details on managing metrics, see [Purge the metrics database](/docs/apimanager_analytics/metrics_db_purge).

## Next steps

After you ensured that you have all prerequisites in place, you can [setup a basic environment](/docs/amplify_analytics/op_insights_config_elastic_singlenode) to test the solution in a development environment.

---
title: Amplify Analytics Operational Insights
linkTitle: Operational Insights
weight: 70
date: 2022-06-09
hide_readingtime: true
description: Configure API Gateway with Elasticsearch to observe millions of requests across different API Gateway instances.
---

Amplify Analytics solution helps you to leverage information about your APIs usage to not only manage your technology infrastructure and operations better, but also to generate additional insights for your businesses. For more information on Amplify Analytics solution, see **LINK**.

Operational Insights is part of the Amplify Analytics solution. It works on top of the analytics stack, [Elasticsearch, Logstash, and Kibana](https://www.elastic.co/elasticsearch/) (ELK), and its advantage over [API Gateway Traffic Monitor](/docs/apimanager_analytics/analytics_intro/) is that it allows you to observe millions of requests across different API Gateway instances in a long time frame.

Operational Insights uses Elasticsearch as the data source (instead of OPSDB) for API Gateway built-in [Traffic Monitor](/docs/apimanager_analytics/analytics_intro/), which resolves the performance and scalability challenges with OPSDB.

![API Gateway Analytics](/Images/concepts/reporter.png)

API Management Operational Insights components provides the following key benefits:

**Performance**:

When having many API Gateway instances with millions of requests, the API Gateway traffic monitor (link to classic monitor in our docs) can become slow and the observation period quite short. Operational insights solve that performance issue, and make it possible to observe a long time-frame and get other benefits by using a standard external datastore, the Elasticsearch

**Visibility**:

The solution allows API Manager users to use the standard traffic monitor and see only the traffic of their own APIs. This allows API service providers who have registered their APIs to monitor and troubleshoot their own traffic without the need for a central team.

**Analytics**:

Deliver standard dashboards that provide analysis capabilities across multiple perspectives. It also allows you to add your own dashboards.

## High level steps to use Operational Insights

The following is a summary of the high level steps to use Operational Insights:

* Ensure that you have read the prerequisites section (**link**).
* Size your infrastructure (**link**).
* Configure your system for a single node Elasticsearch cluster (**link**).
* Configure your API Gateway Manager with Elasticsearch (**link**).
* Configure your system for a production environment (**link**).

## Monitoring and reporting with API Gateway Analytics

When you have configured the metrics database and API Gateway Analytics, you can start monitoring your API traffic and generating reports in API Gateway Analytics. For details, see [Get started with API Gateway Analytics](/docs/apimanager_analytics/analytics_start/).

This documentation describes how to set up and manage your metrics database, how to configure API Gateway, and how to use Operational Insights to monitor your API Gateway domain.

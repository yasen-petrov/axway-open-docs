---
title: Configure multiple API Managers
linkTitle: Configure multiple API Managers
weight: 20
date: 2022-09-19
description: Create a mapping group to use multiple API Managers with Operational Insights.
---

If you have several API Managers within your domain, you must configure a mapping of which group (`groupId`) belongs to which API Manager. The group ID represents the domain group and is attached to each Open Traffic or Metric Event. It is used by Logstash to send the request to the API Builder Lookup API, which uses it to perform the lookup against the relevant API Manager.

The following syntax is used to configure a mapping of the group:

```bash
API_MANAGER=group-2|https://api-manager-1:8075, group-5|https://api-manager-2:8275
```

```bash
# helm example update the apimgrUrl variable in the values.yaml:
apibuilder4elastic:
  apimgrUrl: "group-2|https://api-manager-1:8075, group-5|https://api-manager-2:8275"
```

In this example, all events of `group-2` are enriched with the help of the API Manager (<https://api-manager-1:8075>) and of `group-5` accordingly with <https://api-manager-2:8275>.

### Configure different topologies and domains

You can use the component with different domains and topologies. An example is the different hubs (US, EMEA, APAC, and son on) as the following image shows.

![Example](/Images/op_insights/op_insights_index_per_region.png)

The different hubs each have their own Admin Node Manager and API Manager, but you still must store all API events in a central Elasticsearch instance. For this purpose, the configurable `GATEWAY_REGION` in Filebeat is used. If this region is configured (for example, `US-DC1`), all documents from this region are stored in separated indices, which enables global analytics in the Kibana dashboards

Also, in this case, the API Managers must be configured accordingly with the Region and GroupID of the event. See the following sections with examples for Helm and for Docker Compose.

#### Docker Compose example

Set the API MANAGER variable in the `.env` file to multiple API Managers:

```bash
API_MANAGER=https://my-apimanager-0:8075, group-1|https://my-api-manager-1:8175, group-5|https://my-api-manager-2:8275, group-6|US|https://my-api-manager-3:8375, group-6|eu|https://my-api-manager-4:8475
```

#### Helm example

Update the `apimgrUrl` variable in the `values.yaml`:

```bash
apibuilder4elastic:
  apimgrUrl: "https://my-apimanager-0:8075, group-1|https://my-api-manager-1:8175, group-5|https://my-api-manager-2:8275, group-6|US|https://my-api-manager-3:8375, group-6|eu|https://my-api-manager-4:8475"
```

In this example, API Managers are configured per Region and GroupID. So, if an event is processed, which has a Region and GroupID matching the configuration, then the configured API Manager is used. This includes the lookup for the API details as well as the user lookup for the authorization. If the region does not fit, a fallback is made to a group and  to the generally stored API manager.

Configuring per region only is not possible.

{{< alert title="Note" color="primary" >}}After API Builder is started, a login to each API Manager is performed to validate the configuration. Currently, the same API Manager user (`API_MANAGER_USERNAME`/`API_MANAGER_PASSWORD`) is used for each API Manager.{{< /alert >}}
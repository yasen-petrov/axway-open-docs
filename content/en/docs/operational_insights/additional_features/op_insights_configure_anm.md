---
title: Manually configure the Admin Node Manager policy
linkTitle: Manually configure the Admin Node Manager policy
weight: 60
date: 2022-08-12
description: 
---

We recommend that you use the provided policy fragment, `policy-use-elasticsearch-api-7.7.0.xml`, to automatically configure your Admin Node Manager policy, but if you wish to setup the policy manually, follow this section.

## Create the policy manually

Create a new policy and name it **Use Elasticsearch API**. This policy will decide on what API calls can be routed to Elasticsearch.

The following image shows how the policy looks like:

![ANM Policy Snippet](/Images/op_insights/op_insights_node_manager_use_es_api.png)

## Extract query parameters into attributes

The **Extract REST Request Attributes** filter is used to extract given REST API query parameters into attributes, which is required to get the optional parameter, `useOpsdb`, which can be used to skip Elasticsearch and use the internal OpsDB.

![Extract REST Attributes](/Images/op_insights/op_insights_extract_rest_attributes.png)

## Skip Elasticsearch

The **Compare Attribute** filter is used to check whether the `useOpsdb` parameter is set to `true`. If `true`, Elasticsearch is not used to handle this request.

![Skip Elasticsearch](/Images/op_insights/op_insights_skip_elasticsearch_useOpsdb.png)

To make use of this optional parameter, you must configure it in your `<apigateway>/config/acl.json` file as an allowed parameter.

```bash
"ops_get_messages" : { "path" : "/ops/search?protocol=&format=&from=&count=&order=&rorder=&ago=&field=&value=&op=&jmsPropertyName=&jmsPropertyValue=&useOpsdb=" },
```

If you do not configure this parameter, the ANM will return a `403` error.

After enabling the parameter and forcing the use of OpsDB, you must send a request to the ANM Traffic Monitor. For example:

```bash
https://admin-nodemanager:8090/api/router/service/instance-1/ops/search?useOpsdb=true
```

## Check whether endpoints are managed by Elasticsearch API

The **Compare Attribute** filter, named **Is managed by Elasticsearch API?**, checks whether the requested API can be handled by API Builder ElasticSearch Traffic Monitor API for each endpoint based on the attribute `http.request.path`.

As a basis for decision-making, a criteria for each endpoint needs to be added to the filter configuration.

The following endpoints are currently supported by API Builder based on the Traffic Monitor API:

| Endpoint       | Expression               | Comment |
| :---          | :---                 | :---  |
| **Search**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/search$` | Provides the data for the HTTP traffic overview and all filtering capabilities.|
| **Circuitpath**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/stream\/[A-Za-z0-9]+\/[^\/]+\/circuitpath$` | Provides the data for the filter execution path as part of the detailed view of a transaction.|
| **Trace**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/trace\/[A-Za-z0-9]+[\?]?.*$` | Returns the trace information and the `getinfo` endpoint, which returns the request detail information including the HTTP header of each leg.|
| **GetInfo**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/[A-Za-z0-9]+\/[A-Za-z0-9]+\/[\*0-9]{1}\/getinfo[\?]?.*$` |Provides information for the Request-Response-Details.|
| **Payload**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/stream\/.*\/\d+\/(?:sent|received)$` |Payload endpoint.|

The following image shows how the **Compare Attribute** filter looks like:

![ANM Policy Snippet](/Images/op_insights/op_insights_isManagedbyElasticsearchAPI.png)

## Set region filter

The **Set Attribute** filter, named **Set region filter**, creates a new attribute: `regionFilter`, which is used during the connection to restrict the result based on the region of the Admin Node Manager. It works by way of the environment variable, `env.REGION`. This setup is optional.

![Set Region Filter Details](/Images/op_insights/op_insights_setRegionFilter.png)

Example:

```bash
region=${env.REGION == '[invalid field]' ? "" : env.REGION}
```

## Add region filter

The **Scripting** filter, using Javascript, adds the **Region** filter, which is optional to the `http.request.rawURI` attribute.

```javascript
function invoke(msg) {
    var httpRequestRawURI = msg.get("http.request.rawURI");
    var regionFilter = msg.get("regionFilter");

    if (httpRequestRawURI.contains('?')) {
        httpRequestRawURI += "&" + regionFilter;
    } else {
        httpRequestRawURI += "?" + regionFilter;
    }
    msg.put("http.request.rawURI", httpRequestRawURI);
    return true;
}
```

## Connect to Elasticsearch API

The URL of the **Connect to URL** filter points to your running API Builder docker container and port, which defaults to `8889`, using the `API_BUILDER_URL` environment variable. Additionally, the URL is forwarding the optional region filter based on the configured `REGION` to ensure the Admin Node Manager loads the correct regional data.

![Connect to Elasticsearch API](/Images/op_insights/op_insights_connect_to_elasticsearch_api.png)

Example:

```
${env.API_BUILDER_URL}/api/elk/v1${http.request.rawURI}
```

## Is not implemented

If a given protocol, such as `Directory`, is not implemented, API Builder will return a `501` error to indicate the request should be handled by the OpsDB.

The following image shows how the **Compare attribute** filter looks like:

![Compare Attribute, Is Not Implemented](/Images/op_insights/op_insights_is_not_implemented.png)

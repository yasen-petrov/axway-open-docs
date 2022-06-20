---
title: Configure Elasticsearch for a single instance
linkTitle: Basic setup
weight: 30
date: 2022-06-09
description: Configure a basic setup for Operational insights to test Elasticsearch in a single instance and get prepare for a deployment in production later.
---

The basic setup explains the individual components, how they can be deployed and work together. This is a fully functional configuration to get you started as fast and as easy as possible, and which allows you to try out Operation insights and gain experience, and be better prepared for the deployment in your production environment.

{{< alert title="Note">}}This configuration is recommended for development environments.{{< /alert >}}

After completing the basic setup you will have a single node Elasticsearch instance including multiple Kibana dashboards. At this point, API Gateway Manager is not yet configured, therefore this single instance receives data from 1 to many API Gateways via Filebeat, Logstash, and API Builder, and is accessible via the Traffic Monitor.

You can also import and use the sample Kibana Dashboard or create your own visualizations.

This basic setup uses minimal parameters to run and test the solution on a machine with at least 16 GB of RAM including the API Management platform (for example, like the Axway internal API Management reference environment). Therefore, this is not suitable for a production environment without further configuration.

For a production environment, check the parameters mentioned in the [env-sample](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648) at the beginning and set them if necessary. (**TO BE REVIEWED**)

## High-level steps to perform a basic configuration

The following are the high-level steps to perform a basic configuration:

* Enable Open Traffic Event Log. For more information, see [Configure open traffic event logging](/docs/apim_administration/apigtw_admin/admin_open_logging#configure-open-traffic-event-logging).
* Download and extract the release package (release package of what? I don't understand the changelog file).
* Setup Elasticsearch. For more information, watch [Axway APIM with Elasticsearch - Setup Single Node Elasticsearch cluster](https://www.youtube.com/watch?v=x-OdAdV2N7I) YouTube video. (**I think that some customers can't access youtube in their companies, is it possible to maybe move this video to AU?**)
* Setup Kibana. For more information, watch [Axway APIM with Elasticsearch - Setup Kibana](https://www.youtube.com/watch?v=aLODAuXDMzY). (**Ditto! maybe we could ask AU to create a series of "Axway APIM with Elasticsearch"**).
* Setup Logstash, API-Builder, and Memcached. For more information, watch [Axway APIM with Elasticsearch - Setup Logstash and API-Builder](https://www.youtube.com/watch?v=lnSjF2tUS8Y).
* Setup Filebeat. For more information, watch [Axway APIM with Elasticsearch - Setup Filebeat](https://www.youtube.com/watch?v=h0AdztZ2bSE).
* Install Kibana dashboards.
* Configure API Gateway Manager

## Set up your basic configuration

The following sections explain how to set up Operation insights with a basic configuration, which you can expand later to configure your production environment

### Enable Open Traffic Event Log

placeholder

### placeholder

placeholder

### Configure API Gateway Manager

Configure your API Gateway Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. By default, the API Gateway Manager API listens to port 8090 for administrative traffic. This API is responsible to serve the Traffic Monitor and it needs to be configured to use the API Builder REST API instead.

To configure API Gateway Manager to render log data from Elasticsearch, follow these steps:

1. Open the API Gateway Manager configuration in Policy-Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
3. To be continued...

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.
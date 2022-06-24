---
title: Configure Elasticsearch for a single instance
linkTitle: Basic setup
weight: 30
date: 2022-06-09
description: Configure a basic setup for Operational Insights to test Elasticsearch in a single instance.
---

The basic setup explains the individual components, how they can be deployed and work together. This is a fully functional configuration to get you started as fast and as easy as possible, and which allows you to try out Operational Insights and gain experience, and be prepared for the deployment in your production environment.

{{< alert title="Note">}}This configuration is recommended for development environments.{{< /alert >}}

After completing the basic setup you will have a single node Elasticsearch instance including multiple Kibana dashboards. At this point, API Gateway Manager is not yet configured, therefore this single instance receives data from 1 to many API Gateways via Filebeat, Logstash, and API Builder, and is accessible via the Traffic Monitor.

You can also import and use the sample Kibana dashboard or create your own visualizations.

This basic setup uses minimal parameters to run and test Operational Insights on a machine with at least 16 GB of RAM including the API Management platform (for example, like the Axway internal API Management reference environment). Therefore, this is not suitable for a production environment without further configuration.

For a production environment, check the parameters mentioned in the [env-sample](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648) and set them if necessary. (**TO BE REVIEWED**)

You can deploy Operational Insights using a Helm chart or Docker Compose. This page is organized in three main sections describing the common setup for both Helm and Docker Compose options, and two specific sections for each Helm and Docker Compose.

## Common setup for Helm and Docker Compose

The following are the common setup for both Helm and Docker Compose options.

### Enable Open Traffic Event Log

You must [enable Open-Traffic Event log](/docs/apim_administration/apigtw_admin/admin_open_logging/#configure-open-traffic-event-logging) for your API Gateway instances to allow your log files to be created by default at `apigateway/logs/opentraffic`. This location is required for when configuring Filebeat later.

To avoid data loss, it is strongly recommended to increase the disk space for the Open Traffic logs from 1 GB to at least 8 GB, particularly if you have a lot of traffic. For example, if you have 100 TPS on one API Gateway instance, depending on your custom policies, the oldest log file will be deleted after approximately 30 minutes with only 1 GB OpenTraffic log configured. If for any reason Filebeat and Logstash are not running to process events for more than 15-20 minutes, you will have a loss of data as it also takes some time to catch up.

### Install Kibana dashboards

To install the Kibana dashboards, use the Kibana Menu **Stack Management > Saved Objects > Import saved objects** and select the following file from the release package, `kibana/dashboards/7/*.ndjson`.

In the **Import options** window, select **Automatically overwrite conflicts**. This is the default setting, and you must always select it regardless of whether you are installing the dashboards for the first time or after an update.

You can create customized visualizations and dashboards, but do not change the existing ones as they will be overwritten with the next update. If you have created your own visualizations and dashboards, they will not be changed by the import.

### Configure API Gateway Manager

Configure your API Gateway Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. By default, the API Gateway Manager API listens to port 8090 for administrative traffic. This API is responsible to serve the Traffic Monitor and it needs to be configured to use the API Builder REST API instead.

To configure API Gateway Manager to render log data from Elasticsearch, follow these steps:

1. Open the API Gateway Manager configuration in Policy-Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
3. To be continued...

### Setup Admin-Node-Manager

Watch this video for an overview: [Traffic-Monitor and Kibana Dashboard](https://youtu.be/OZ0RNnqE6hs),

As the idea of this project is to use the existing API-Gateway Manager UI (short: ANM) to render log data now provided by Elasticsearch instead of the individual API-Gateway instances before (the build in behavior), it is required to change the ANM configuration to make use of Elasticsearch instead of the API-Gateway instances (default setup). By default, ANM is listening on port 8090 for administrative traffic. This API is responsible to serve the Traffic-Monitor and needs to be configured to use the API-Builder REST-API instead.

**TO BE CONTINUED...**

### Traffic-Monitor for API-Manager Users

AAA
S
## Virtual machines (Docker-Compose)

The following sections cover the configuration using Docker Compose.

### Download and extract the release package

Download the relevant ELK package.

The community release always reflects the state of development. Check the [changelog](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/CHANGELOG.md) to ensure that you have select the correct version. For more information, see [Is this component officially supported by Axway](/docs/amplify_analytics/op_insights_faq/#is-this-component-officially-supported-by-axway).

We recommend that you run Operational Insights on different machines. Therefore, download and unpack the release package on each machine.

**Axway supported version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.2.0/axway-apim-elk-v4.2.0.tar.gz -O - | tar -xvz
```

**Community version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.5.0/axway-apim-elk-v4.5.0.tar.gz -O - | tar -xvz
```

To simplify updates, it is recommended to create a symlink folder and rename the provided file `env-sample` to `.env` in each machine as follows:

```bash
ln -s axway-apim-elk-v1.0.0 axway-apim-elk
cd axway-apim-elk-v1.0.0
cp env-sample .env
```

You must store the `.env` file as a central configuration file in a version management system.

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.
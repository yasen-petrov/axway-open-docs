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

### Setup Admin Node Manager

Watch this video for an overview, [Traffic-Monitor and Kibana Dashboard](https://youtu.be/OZ0RNnqE6hs).

Configure Admin Node Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. The Admin Node Manager API listens to port 8090 for administrative traffic by default and itt is responsible for serving the Traffic Monitor. You must now configured node manager to use the API Builder REST API instead.

To configure the node manager to render log data from Elasticsearch, follow these steps:

1. Open the node manager configuration in Policy Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
    Do not use the XML file downloaded directly from the GitHub project as it might contain different certificates.
3. [Update main policy](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/nodemanager) by inserting the "Use Elasticsearch API" policy as a callback policy (see filter, **Shortcut filter**) into the main policy, **Protect Management Interfaces**, and wire it like shown in the following image.
    **IMAGE**
4. Disable the audit log for **Failure** transactions to avoid not needed log messages in the node manager trace file.
5. Open the `apigateway-install-dir>/apigateway/conf/envSettings.props` file and add the following new environment variable, `env.API_BUILDER_URL=https://apibuilder4elastic:8443`.
6. If you are using [multiple regions](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#different-topologiesdomains), you must configure the appropriate region to restrict the data of node manager to the correct regional data, for example, `env.REGION=US`.
7. Copy the new configuration from the Policy Studio project folder (path on Linux, `/home/<user>/apiprojects/\<project-name\>`) back to the Admin Node Manager folder (`\<install-dir\>/apigateway/conf/fed`).
8. Restart the Admin Node Manager.

### Traffic Monitor for API Manager users

AAA

## Setup for Helm

The following sections cover the configuration specific for using Helm charts.

placeholder

## Setup for Docker Compose

The following sections cover the configuration specific for using Docker Compose (virtual machines).

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
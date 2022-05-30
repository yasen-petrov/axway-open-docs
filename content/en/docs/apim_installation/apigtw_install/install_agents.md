{
"title": "Install Discovery and Traceability agents",
"linkTitle": "Install Discovery and Traceability agents",
"weight":"20",
"date": "2022-05-12",
"description": "Prerequisites and steps for installing Discovery and Traceability agents."
}

The [Discovery agent](https://docs.axway.com/bundle/amplify-central/page/docs/connect_manage_environ/connected_agent_common_reference/category_mapping/index.html) is used to discover new published APIs or any updated APIs. After the APIs are discovered, they are published to Amplify Central.

The [Traceability agent](https://docs.axway.com/bundle/amplify-central/page/docs/connect_manage_environ/connected_agent_common_reference/traceability_usage/index.html) is used to prepare the transaction events that are sent to Amplify platform. Each time an API is called by a consumer, an event (summary and details) is sent to Amplify Central and is visible in Business Insights.

## Prerequisites

* Ensure that all of the prerequisites detailed in [API Gateway System requirements](/docs/apim_installation/apigtw_install/system_requirements) are met.
* You have a valid pair of private and public keys to connect with Amplify Central, as detailed in [Agents configuration](https://docs.axway.com/bundle/amplify-central/page/docs/connect_manage_environ/connect_api_manager/gateway-administation/index.html).

## Install in GUI mode

To install the Discovery and Traceability agents in GUI mode along with API Gateway, follow the steps described in [API Gateway installation](/docs/apim_installation/apigtw_install/installation), using the following options:

* Select the **Custom** setup type.
* Select to install the following components:
    * Discovery and Traceability agents

## Install in unattended mode

The following example shows how to install the Discovery and Traceability agents in unattended mode:

```
./APIGateway_7.7_Install_linux-x86-32_BN<n>.run --mode unattended --setup_type advanced --enable-components agentsConfiguration --disable-components apimgmt,apigateway,nodemanager,cassandra, qstart,policystudio,configurationstudio,analytics,apitester,packagedeploytools
```

For more information see [API Gateway unattended installation](/docs/apim_installation/apigtw_install/installation_unattended).

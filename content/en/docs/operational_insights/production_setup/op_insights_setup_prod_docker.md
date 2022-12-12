---
title: Configure a production setup for Docker compose
linkTitle: Production setup for Docker Compose
weight: 20
date: 2022-07-20
description: Configure a production setup using Docker Compose to run Operational Insights in a highly available environment.
---

This section covers advanced configuration topics that are required for a production environment deployed with Docker compose.

## Before you start

* Ensure that you have all [prerequisites](/docs/operational_insights/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](docs/operational_insights/basic_setup).

## Considerations to use Elasticsearch for multi-nodes

For a production environment, Elasticsearch must run in a multi-node Elasticsearch cluster environment. Indices are configured so that available nodes are automatically used for primary and replica shards. If you use only one Elasticsearch node, the replica shards cannot be assigned to any node, which causes the cluster to remain in `Yellow` state and results in some tasks not being performed by Elasticsearch, for example, lifecycle management of the indexes.

You can setup a multi-node Elasticsearch cluster by changing the default settings of the `ELASTICSEARCH_HOSTS` parameter, located in your `.env` file.

Before changing the `ELASTICSEARCH_HOSTS` parameter, consider the following:

* The default HTTP ports for the three Elasticsearch instances (elasticsearch1, elasticsearch2, elasticsearch3) are `9200`, `9201`, and `9202`. The default transport ports are `9300`, `9301`, and `9302`. These ports are exposed by Docker Compose through the Docker containers.
* Based on the HTTP ports, the transport port is derived, for example, `9201` --> `9301`.
* Other ports are available and can be configured in your `.env` file.
* Based on the specified URLs, the necessary Elasticsearch parameters are set when creating or starting the Elasticsearch Docker containers.
* Multiple Elasticsearch nodes are only useful if they run on different hosts, and assuming that the release package is downloaded on each individual host and your configured `.env` file is provided.
* You can always add more nodes to the Elasticsearch cluster to provide additional disk space and computing power. You can start with two nodes, and add another cluster node at any time later.
* All clients (Filebeat, Logstash, API Builder, and Kibana) use the Elasticsearch hosts specified in the `.env` file. This means that you do not need a load balancer in front of the Elasticsearch cluster to achieve high availability at this point.
* Ensure all clients are configured to use the available Elasticsearch hosts by restarting the client containers after changing the `.env` file. This ensures the latest changes to the `ELASTICSEARCH_HOSTS` parameter is used by the client containers.

{{< alert title="Note" >}}If you are using an external Elasticsearch cluster, you can configure the HTTP and transport ports being used by your Elasticsearch cluster in your `.env` file. {{< /alert >}}

## Setup Elasticsearch in multi-nodes

Follow these steps to setup Elasticsearch in a multi-node environment by creating new Elasticsearch nodes to be used within the cluster.

1. Setup the cluster nodes. Operational Insights is prepared to work with five nodes by default, but it can be extended to work with more nodes if necessary. To configure multiple hosts, add the following to your `.env` file:

    ````bash
    ELASTICSEARCH_HOSTS=https://elasticsearch1:9200,https://elasticsearch:9200:9201
    ````

    Run the following command if you wish to change the name of the cluster:

    ```bash
    ELASTICSEARCH_CLUSTERNAME=axway-apim-elasticsearch
    ```
    If you need to explicitly change the host configuration that is used by each node, use the `ELASTICSEARCH_PUBLISH_HOST<n>`, `ELASTICSEARCH_HOST<n>_HTTP`, and `ELASTICSEARCH_HOST<n>_TRANSPORT` parameters.

2. Bootstrap the cluster. We recommend starting one node after the next. The first node sets up the cluster and bootstrap it. Run the following command to start the first cluster node:

    {{< alert title="Note" >}}You can skip this step if you have already initialized your Elasticsearch cluster. This is done in the [Setup Elasticsearch](/docs/operational_insights/basic_setup/#setup-elasticsearch) section of the basic setup for Docker Compose. {{< /alert >}}

    ```bash
    docker compose --env-file .env -f elasticsearch/docker compose.es01.yml -f elasticsearch/docker compose.es01init.yml up -d
    ```

     This node automatically becomes the master node.

3. Add additional nodes. You can add cluster nodes at any time to increase available disk space or CPU performance. To achieve resilience, it is strongly recommended to set up three cluster nodes to also perform maintenance tasks such as updates. For more information see, [Designing for resilience](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html). It is also possible to have two Elasticsearch nodes running on the same machine. To add a cluster node, configure the `ELASTICSEARCH_HOSTS` parameter in your `env` file to include the new cluster node host, then run the following command:

    ```bash
    # To add for instance a third node, `ELASTICSEARCH_HOSTS` must contain three nodes
    docker compose --env-file .env -f elasticsearch/docker compose.es03.yml up -d
    ```

    Verify the new node was successfully added to the cluster and shards are assigned to it via Kibana.

4. Restart the clients (Kibana, Filebeat, Logstash, API Builder) to ensure they have all relevant Elasticsearch nodes available for use in the case of node downtime.

## Configure cluster UUID

This step is optional, but it is required if you wish to monitor your Filebeat instances as part of the stack monitoring.

1. Obtain the cluster universally unique ID (UUID) by opening the following in your browser, <https://elasticsearch1:9200/>.
2. (Optional) Make your Filebeat instances unique by configuring the following parameters in your `env` file:

    * `GATEWAY_NAME`
    * `GATEWAY_REGION`

3. Restart Filebeat service to activate the changes.

## Configure API Manager

Before a document is sent to Elasticsearch, additional information for the processed API is requested by Logstash from the API Manager through an API lookup. This lookup is handled by API Builder and performed against the configured API Manager. By default, the `API_MANAGER` parameter is not used, and the configured Admin Node Manager host is used instead to configure the APIManager host. For example,

```none
API_MANAGER=https://my.apimanager.com:8075
```

To configure multiple API Managers, see [Multiple API Managers](/docs/operational_insights/op_insights_multiple_apimanagers/).

## Support for custom properties

Operational Insights supports configured API Manager API custom properties by default. This means that the custom properties are indexed within the `customProperties` field, in Elasticsearch, and they can be used for customer-specific evaluations.

It is also possible to index runtime attributes (policy attributes) in Elasticsearch and analyze them in Kibana, for example.

The following sections describe how to configure custom properties.

### Export the custom attributes

In Policy Studio, configure which attributes should be exported to the transaction event log. For more information on how to configure messages attributes to be stored in transaction events, see [Transaction event log settings](/docs/apim_reference/log_global_settings#transaction-event-log-settings).

All exported attributes are found in the Elasticsearch indices: `apigw-traffic-summary` and `apigw-hourly-traffic-summary` within the `customMsgAtts` field.

### Configure custom properties

You must configure your attributes before you can use them in aggregations for reports later on. By way of the `EVENTLOG_CUSTOM_ATTR` parameter, you can define which attributes should be indexed in Elasticsearch. It is important that the fields have as high cardinality as possible to work efficiently in aggregations. The fields are indexed as a keyword in Elasticsearch. It is also possible to index fields as text, but these are not available in the long-term data, and therefore, are not recommended.

After restarting APIBuilder4Elastic, Elasticsearch (index templates and transform job) is then configured accordingly to the `EVENTLOG_CUSTOM_ATTR` parameter. It takes around four hours before the custom properties are available in the transformed data (`apigw-hourly-traffic-summary`).

## Activate user authentication

Follow these steps to set up authentication and authorization for Elasticsearch and Kibana.

1. Generate built-in user passwords.

    This step can be ignored if you are using an existing Elasticsearch cluster. Elasticsearch is initially configured with a number of built-in users that do not have a password by default. The first step is to generate passwords for these users. It is assumed that the following command is executed on the first `elasticsearch1` node:

    ```bash
    docker exec elasticsearch1 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url https://localhost:9200"
    ```

    As a result of this command, you will see the randomly generated passwords for the following users:

    * apm_system
    * kibana_system
    * kibana
    * logstash_system
    * beats_system
    * remote_monitoring_user
    * elastic

2. Setup passwords. You must update your `.env` file and set up all passwords for the previously listed users. If you are using an existing Elasticsearch cluster, you must use the passwords provided to you. The `.env` file contains information about each password and what it is used for. You can start by using the `elastic` user account to login and then create individual users and permissions as follows:

    ```bash
    BEATS_SYSTEM_USERNAME=beats_system
    BEATS_SYSTEM_PASSWORD=<GENERATED_BEATS_PASSWORD>
    
    KIBANA_SYSTEM_USERNAME=kibana_system
    KIBANA_SYSTEM_PASSWORD=<GENERATED_KIBANA_PASSWORD>
    
    LOGSTASH_SYSTEM_USERNAME=logstash_system
    LOGSTASH_SYSTEM_PASSWORD=<GENERATED_LOGSTASH_PASSWORD>
    
    LOGSTASH_USERNAME=elastic
    LOGSTASH_PASSWORD=<GENERATED_ELASTIC_PASSWORD>
    
    API_BUILDER_USERNAME=elastic
    API_BUILDER_PASSWORD=<GENERATED_ELASTIC_PASSWORD>
    ```

3. Disable anonymous user. In the `.env` file, uncomment the following line:

    ```bash
    ELASTICSEARCH_ANONYMOUS_ENABLED=false
    ```

    After restarting, Kibana will prompt you to login before continuing.

4. Enable user authentication in Kibana. In your `.env` file, uncomment the following line to enable user authentication in Kibana:

    ```bash
    KIBANA_SECURITY_ENABLED=true
    ```

5. Restart the services. After you have configured all passwords and configured the security parameters, restart all services.

    * **Filebeat** - It will now use the `beats_system` user to send monitoring information.
    * **Logstash** - It will now use the `logstash_system` user to send monitoring data and `logstash` user to insert documents.
    * **Kibana** - It will now use the `kibana_system` user to send monitoring data.
    * **API Builder** - It will now use the `elastic` user to query Elasticsearch.

    It is important to set up a dedicated API Builder user, instead of using the `elastic` super-user. The monitoring users are used to send metric information to Elasticsearch to enable stack monitoring, which gives you insights on event processing of the complete platform.

## Configure the retention period

Because new data is continuously stored in Elasticsearch in various indexes, these must be removed after a certain period of time to free up space in your container. The Elasticsearch [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) (ILM) feature is used for this purpose. It defines different lifecycle stages per index. The ILM policies are automatically configured with default values using configuration files and can be reviewed in Kibana. You can also configure the lifecycle of the data accordingly to your requirements. The indices pass through stages such as `Hot`, `Warm`, and `Cold`, which can be used to deploy different performance hardware per stage. This means that traffic details from, for example, two weeks ago, no longer have to be stored on high-performance machines.

The configuration is defined per data type (Summary, Details, Audit, and so on), and the following table shows an overview of the default values. The number of days that is crucial for the retention period is the `Delete` days, which provides the number of days that the data is guaranteed to be available.

| Data type              | Description                                                            | Hot (Rollover) | Warm    | Cold    | Delete  |
| :---                   |:---                                                                    | :---           | :---    | :---    | :---    |
| **Traffic-Summary**    | Main index for Traffic Monitor overview and primary dashboard          | 30GB / 7d      | 0d      | 5d      | 10d    |
| **Traffic-Details**    | Details in Traffic Monitor for policy, headers, and payload reference   | 30GB / 7d      | 0d      | 5d      | 10d    |
| **Traffic-Trace**      | Trace messages belonging to an API request shown in Traffic Monitor    | 30GB / 7d      | 0d      | 5d      | 10d    |
| **General-Trace**      | General trace messages, like `Start-` and `Stop-Messages`                    | 30GB / 7d      | 0d      | 5d      | 10d    |
| **Gateway-Monitoring** | System status information (CPU, HDD, etc.) from Event files            | 10GB / 30d     | 0d      | 50d     | 100d    |
| **Domain-Audit**       | Domain Audit Information, as configured in Admin Node Manager           | 10GB / 30d     | 0d      | 300d    | 750d    |

{{< alert title="Warning" color="warning" >}}Deleted data cannot be recovered.{{< /alert >}}

Before you configure how long the indexed data should be kept in Elasticsearch, ensure to read and understand the following information thoroughly, because once deleted, data cannot be recovered:

* Individual API transactions are stored as documents in Elasticsearch indices. However, ILM operates on a per-index basis. That is, the entire index is either deleted or kept alive. Data is not deleted on a per-document basis. Therefore, you can only control the retention period for an entire index, not per document.
* When API transactions are stored in an index, the size of the index increases accordingly. To prevent an index from growing infinitely, it can be rolled over after a certain time. A new active index is created, which is used to write the data. This replaces the old index, which is only used for reading. This process is called [rollover](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-rollover.html). To avoid having to control this process manually, you can use ILM policies in Elasticsearch. These ILM policies perform rollovers based on defined rules and then send the index through further phases for various purposes.
* These ILM policies are configured automatically by the solution with default values and are stored and managed for each index in Elasticsearch. The default values result in the data being available for at least two weeks.

If you would like to customize the lifecycle, then you can provide a corresponding configuration file and use the `RETENTION_PERIOD_CONFIG` parameter. This is used to adapt the ILM policies accordingly.

Here is an example:

```json
{
    "retentionPeriods": {
        "apigw-traffic-summary": {
            "rollover": {
                "max_age": "7d",
                "max_size": "15gb"
            }, 
            "retentionPeriod": "7d"
        }, 
        "apigw-traffic-details": {
            "rollover": {
                "max_age": "7d",
                "max_size": "15gb"
            }, 
            "retentionPeriod": "6d"
        }, 
        "apigw-traffic-trace": {
            "rollover": {
                "max_age": "7d",
                "max_primary_shard_size": "15gb"
            }, 
            "retentionPeriod": "5d"
        }
    }
}
```

The configuration is defined per index and is divided into two areas: when the rollover should happen and how many days after the rollover the data should still be available.

The following steps show how to configure the retention period.

1. Create a new file for your retention period configuration. For example,`config/custom-retention-period.json`. You can use the `config/my-retention-period-sample.json` file as a template.
2. Define the rollover period. Note that the time period until the rollover of an index is not exactly fixed. For example, if you specify a maximum age and size for an index, then the index will be rolled over as soon as a condition is met.

    * If the maximum size is too small for your transaction volume, then an index can meet the size condition in less than 24 hours and will be rolled over.
    * If the maximum size is too large, the index will be rolled when it reaches the maximum age (for example, after 7 days).

    So how long the data is available from the very beginning to the end of an index is the sum of the period from the index's initial creation to the rollover plus the period until the delete. As the rollover date cannot be defined exactly, you need to monitor your system accordingly and adjust the lifecycle accordingly to get the desired retention time.

    You can use the following conditions for the rollover:

    * **max_age**: Defines the maximum age of an index until it is rolled over.
    * **max_size**: The maximum index size. As every index has a primary and replica shard, the required disk space allocated must be doubled (max_size: 30 GB requires 60 GB disk space).
    * **max_primary_shard_size**: Starting with an Elasticsearch version `7.13`, you can also define the maximum shard size of an index. All indexes, except `apigw-management-kpis` and     `apigw-domainaudit`, have five shards. So you have to multiply the specified size by 5.

    For more information, see [ILM Rollover options](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-rollover.html#ilm-rollover-options).

3. Define the retention period. Use the `retentionPeriod` property on your configuration file to define the time period for which the data is guaranteed to be available. As already described, the time until the rollover of the index adds to this. You can only specify days.

4. Apply the configuration. Use the `RETENTION_PERIOD_CONFIG=./config/custom-retention-period.json` parameter from your `.env` file to reference your retention configuration file, then restart API Builder as follows:

    ```bash
    docker compose stop apibuilder4elastic docker compose up
    ```

    You can check in Kibana whether the ILM policy has been adjusted accordingly. In Kibana, click **Stack Management > Index Lifecycle Policies**, then open the corresponding policy and check the phase.

### Recommended environment

[Helm](/docs/operational_insights/update_operational_insights/op_insights_update_helm/) is considered the targeted deployment methodology for this component and many security defaults have been set for this method of deployment. In a Docker Compose environment, you must take the following security checklist into account:

* Setting a healthcheck for each container
* Setting memory limits for each container
* Setting CPU requirements for each container
* Configuring a restart policy for each container
* Restricting container capabilities.

### Further notes

Observe the following items when deploying the component in a Docker Compose environment:

* Changes to the ILM policy have no effect on indices that have already been rolled over, as these have already entered lifecycle management.
* Indexes should not be too small, as this increases the load on Elasticsearch too much.
    * For each active index there are five Primary and five Replica-shards.
    * Each shard corresponds to a Lucene instance, which consumes corresponding resources.
    * The smaller an index, the more indexes, the more shards, the more resources are needed.
    * The Elastic's recommendation size is 30 GB. Operational Insights does not allow index size below 5 GB.
* It is optional to use different hardware per stage.
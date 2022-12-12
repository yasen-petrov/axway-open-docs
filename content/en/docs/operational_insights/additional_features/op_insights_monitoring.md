---
title: Configure Metricbeat for monitoring and metrics
linkTitle: Configure Metricbeat
weight: 50
date: 2022-08-05
description: Configure Metricbeat for monitoring and metrics in Operational Insights component deployed either in a Docker Compose environment or in Helm charts.
---

Operational Insights component is internally configured for self-monitoring by default. This means that Logstash, Kibana, Filebeat, and so on, independently send monitoring information (metrics) to Elasticsearch. However, this approach is not recommended by Elastic and is deprecated. Metricbeat should be used instead. However, the component cannot easily be delivered with a pre-configured Metricbeat, because this depends too much on the deployment, so to use Metricbeat you must set some parameters in your `.env` file.

This section covers how to enable Metricbeat and monitor the running Docker containers in addition to the pure Elastic stack.

### Activate Metricbeat

The first step to enable Metricbeat is to activate it.

* Check that the `METRICBEAT_USERNAME` and `METRICBEAT_PASSWORD` user is set up correctly. This user must have rights to Kibana (to upload dashboards) and Elasticsearch (to create indexes).
* Set the `METRICBEAT_ENABLED=true` parameter. This will be populated when the Metricbeat container starts.
* Set the `SELF_MONITORING_ENABLED` parameter to `false` to disable legacy monitoring.

### Configure Metricbeat

Use the `METRICBEAT_MODULES` parameter, which must be set differently for each host, depending on which services are running on which host.

| Component              | Description                           |
| :---                   | :---                                  |
| **elasticsearch**      | Replaces internal monitoring so that Elasticsearch metrics appear in Kibana stack monitoring. **Important**: One Metricbeat monitors all Elasticsearch nodes based on the `ELASTICSEARCH_HOSTS` parameter, and Kibana if needed.                                   |
| **kibana**             | Enables monitoring of Kibana analogous to Elasticsearch. You can use a Metricbeat for monitoring Elasticsearch and Kibana as explained. |
| **logstash**           | Provides the data in Kibana stack monitoring. |
| **filebeat**           | Provides the data in Kibana stack monitoring. |
| **memcached**          | Captures statistics from Memcached. It is indexed, but currently not used in any dashboard and can be disabled if not needed. |
| **system**             | Provides system metrics such as disk IO, network IO, and so on, in the default configuration, out of the Docker container, which limits the process view. |
| **docker**             | Provides information about running Docker containers that can be displayed in dashboards. Docker containers are automatically detected on the host. |

Set the `METRICBEAT_NODE_NAME` parameter to a descriptive name, which will be displayed later in the Kibana dashboards where the host Metricbeat is running on.

### Start Metricbeat

Start the Metricbeat container on each host:

```bash
docker compose --env-file .env -f metricbeat/docker compose.metricbeat.yml up -d
```

As a result of this command, a container `metricbeat` is started on each host with the given configuration in the belonging `.env` file.

### Disable self-monitoring

Ensure the `SELF_MONITORING_ENABLED` parameter is set to `false`. Then, stop Elasticsearch, Kibana, Logstash, and Filebeat services and restart them with docker compose that they are no longer using self-monitoring. A docker restart is not sufficient here. For example:

```bash
docker stop kibana
docker compose --env-file .env -f kibana/docker compose.kibana.yml up -d
```

{{< alert title="Note" color="primary" >}}
Before restarting an Elasticsearch Node, ensure the cluster state is green to stay online during the restart.
{{< /alert >}}

## Enable Elastic application performance monitoring

You can enable Application Performance Monitoring (APM) to monitor APIBuilder4Elastic and other services (for example, API Builder services).

Elastic [APM](https://www.elastic.co/observability/application-performance-monitoring) allows you to monitor the APIBuilder4Elastic better than just using logs directly from the API Builder. Of course, you can also add other API Builder services or other applications.

After APM is set up, in your running Kibana service, you can use **Kibana > Observability > APM** to access a series of application performance dashboards.

APIBuilder4Elastic is prepared to use an external APM server or to deploy directly with Operational Insights. You can enable the APM service via Docker Compose in your `.env` file or in the Helm chart. The activation is divided into two steps, the setup and start of the APM server and the connection to the APIBuilder4Elastic. The setup step is not necessary if you are using an existing external APM server.

## Activate APM in a Docker Compose deploy

Follow these steps to activate APM:

1. Start the APM server. This launches the APM service as a Docker container. The configured Elasticsearch hosts (`ELASTICSEARCH_HOSTS`) and certificates are used to connect to Elasticsearch. If you want to have user login enabled, you also need to set the `APM_USERNAME` and `APM_PASSWORD` parameters. Ensure that the initial setup user have rights to create templates (for example, the elastic user).

    ```bash
    docker compose --env-file .env -f apm/docker compose.apm-server.yml up -d
    ```

    This starts the APM service as a Docker container:

    ```none
    CONTAINER ID   IMAGE                                                  COMMAND                  CREATED        STATUS                      PORTS                                            NAMES
    cdffd8117b8b   docker.elastic.co/apm/apm-server:7.15.2                "/usr/local/scripts/…"   16 hours ago   Up 16 hours             0.0.0.0:8200->8200/    tcp                           apm-server
    ```

2. To make use of the APM service in APIBuilder4Elastic, you must set the `APM_ENABLED` parameter to `true` in your `.env` file. This will cause the API Builder process to attempt to connect to the configured APM server. During the startup of API Builder, at the very beginning, the following will be logged:

    ```bash
    Application performance monitoring enabled. Using APM-Server: https://axway-elk-apm-server:8200
    ```

3. Activate APM service in APIBuilder4Elastic. If no APM service is specified, then, the default is used (<https://apm-server:8200>). However, you can configure it yourself using the `APM_SERVER` parameter.

## Activate APM in Helm

If you are deploying Operational Insights with Helm, follow this section:

1. Start the APM server in your Helm chart. In your `local-values.yml` activate the APM server. Additionally, set the Elasticsearch cluster UUID to include the APM server in stack monitoring:

    ```bash
       apm-server:
         enabled: true
         elasticsearchClusterUUID: 3hxrsNg6QXq2wSkVWGTD4A
    ```

2. Activate APM service in APIBuilder4Elastic in your Helm chart with the following parameters:

    ```bash
    apibuilder4elastic:
      apmserver:
       # If you want to use an external APM-Server, you can configure the URL here. 
       # Defaults to the internal APM-Server service if enabled
       serverUrl: {}
       # The Certificate-Authory to validate the APM-Server certificate
       # serverCaCertFile: "config/certificates/my-apmserver-ca.crt"
       # You may disable the certificate check
       # verifyServerCert: "false"
    ```

## Disk usage monitoring

It is important that you monitor the disk usage of the Elasticsearch cluster and get alarmed accordingly. Elasticsearch also independently monitors disk usage against pre-configured thresholds and closes write operations when the high disk watermark index is exceeded. This means that no more new data can be written.

To avoid this condition, your alerts should already warn below the Elasticsearch thresholds. The thresholds for Elasticsearch:

* Low watermark for disk usage: 85%.
* High watermark for disk usage: 90%

Your alerts should report a critical alert before 90%. For more information, see [Disk-based shard allocation settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#disk-based-shard-allocation).

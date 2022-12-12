---
title: Update Operational Insights in Docker Compose
linkTitle: Update in Docker Compose
weight: 20
date: 2022-07-28
description: Instructions on how to update Operational Insights component between versions in a Docker Compose environment.
---


The basic principle of Operational Insights is that no files delivered with a release are changed. Only your `*.env` file must be carried over from one release to the next.

This section describes how to update based on Docker Compose deployment. If you have deployed Operational Insights on a Kubernetes cluster using Helm, see [Update Helm](/docs/operational_insights/update_operational_insights/op_insights_update_helm/).

## Zero downtime upgrade

With the following steps you can update the component without downtime. This requires that there are at least two of each Logstash, API Builder, and Memcached configured. All Filebeats must be communicating with all Logstash hosts.

## About components update

The core tool is API Builder, which provides the information about the necessary configuration. In principle, it contains the desired or necessary state suitable for the version, especially about the Elasticsearch configuration, such as index templates, ILM policies, and so on. If the version is updated, API builder checks the current configuration in Elasticsearch and adjusts it if necessary to fit the corresponding version. This includes necessary changes for bug fixes or enhancements.

All tooling associated to Operational Insights play together and only work if they are from the same release. Operational Insights checks if, for example, the index templates have the required version.

It is strongly discouraged to make changes to any files in the project, except the `.env` file and the `config` folder. This is the way to update from one version to the next.

## Upgrade approach

The following lists the overall steps to upgrade Operational Insights:

* Load and unpack the desired release. It is recommended to unpack it next to the existing release.
* Copy your existing `.env` file from the current installation. We recommend using a symlink to a central `.env` file, which should also be versioned if necessary.
* Ensure that you carry over your own certificates into the new release if necessary.
* Depending on which components have changed, the containers must be stopped, then restarted based on the new release with `docker compose up -d`. If, for example, no change is noted in Logstash, then this component does not need to be stopped and restarted, and can continue to run.
* API Builder delivers the configuration for Elasticsearch as well. If API Builder is updated, then it checks on restart and periodically if the Elasticsearch configuration is correct. In addition, API Builder checks whether the Filebeat and Logstash configuration corresponds to the expected version.

## Upgrade command example

The following example shows how to load and unpack a new release and apply your current configuration. Regardless of which components have changed, you must install the same release package on all your machines.

```bash
# Perform these steps on all machines

# Get and extract the release package
wget --no-check-certificate (*weblive link needed*) -O - | tar -xvz
# The following directory has been created
cd axway-apim-elk-v-NEW
# Copy existing configuration from the previous version
cp ~/axway-apim-elk-v-OLD.env .
# Copy existing certificates, scripts, extra config files from the previous version
cp ~/axway-apim-elk-v-OLD/config/all-my-custom-certificates ./config
```

## Upgrade all components

You can now update each component as described in the following sections.

### API Builder, Logstash, Memcached

API Builder, Logstash, and Memcache work as a tight unit and, therefore, it is recommended to stop and update them together. Changes to Logstash do not mean that the Logstash version has changed, but that the Logstash pipeline configuration has changed. Use the new release to start Logstash. The following example shows an update to version `3.2.0`:

```bash
docker compose stop
docker compose up -d
```

Run these commands on all machines running APIBuilder, Logstash, and Memcache.

If you have deployed Operational Insights on a Kubernetes cluster using Helm, see [Configure a basic setup for Helm charts](/docs/operational_insights/basic_setup/op_insights_setup_basic_helm/) for more information.

{{< alert title="Note" >}}When updating the API Builder component, it is important to change the `image` parameter in your `docker-compose.yaml` configuration file to point to the new API Builder image to be used. {{< /alert >}}

### Filebeat

If Filebeat changes with a version, you must update the corresponding configuration on all your API Gateway instances. It is recommended to update Filebeat as the first component, because the Filebeat configuration version is checked by API Builder. If it does not match, Logstash will exit with an error message. Even if you run Filebeat as a native service, you still must copy the configuration (`filebeat/filebeat.yml`) from the release into your configuration.

The following example shows how to update versions:

```bash
docker stop filebeat
docker compose --env-file .env -f filebeat/docker compose.filebeat.yml up -d
```

### Admin Node Manager

Follow the instructions to setup the [Admin Node Manager](/docs/operational_insights/op_insights_basic_setup/#setup-admin-node-manager) based on the most recent Policy fragment shipped with the release. Ensure that the same certificates previously imported into the Admin Node Manager are imported again.

### Dashboards

Follow the  instructions to [import Kibana dashboards](/docs/operational_insights/op_insights_basic_setup/#install-kibana-dashboards) based on the most recent Dashboards shipped with the release. If prompted while importing the new Kibana dashboards, ensure that existing objects are overwritten.

### Parameters

Sometimes, it might be necessary to include newly introduced parameters in your `.env` file, or you might want to use optional parameters. To do this, use the supplied `env-sample` as a reference and copy the desired parameters. Check the [release notes changelog](/docs/apim_reference/allreleaseschangelog/) to know which new parameters have been added.

### Elastic Config

Whether the Elastic configuration has changed, it is for information only and does not require any steps during the update. The required Elasticsearch configuration is built into API Builder and Elasticsearch will be updated accordingly if necessary. This includes the following components:

* Index configuration
* Index templates
* Index lifecycle policies
* Transform jobs

You can find the current configuration in the `apibuilder4elastic/elasticsearch_config` folder.

#### Elasticsearch nodes required

Before proceeding with the upgrade, ensure that your Elasticsearch cluster consists of at least three nodes with the same version. For example, three Elasticsearch nodes running on two machines is perfectly fine for this.

The setup requires three Elasticsearch nodes because there must always be a master node in the cluster. If the master node is stopped, a quorum of remaining cluster nodes must still be running to elect a new master, otherwise an upgraded Elasticsearch node cannot join the cluster. For more information, see [Quorum-based decision making](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-quorums.html)

See [Setup Elasticsearch multi-node](/docs/operational_insights/production_setup/op_insights_setup_prod_docker/#setup-elasticsearch-multi-node) to learn about adding additional cluster nodes. After the upgrade, ensure the Elasticsearch cluster is in `Green` state. You can then remove the third cluster node if required.

## Upgrade steps for Elastic version

It is recommended to run the entire Elastic stack with the same version. When you have a three node Elasticsearch cluster up and running, follow these steps to update the Elastic version.

### Update the .env file

* Open your `.env` file and change the `ELASTIC_VERSION` parameter to the version as specified in the release or the version you would like to use. Ensure that the `.env` file contains the correct, same version on all machines.
* To avoid any downtime, double check that all Elasticsearch clients (API Builder, Logstash, Filebeat) using the `ELASTICSEARCH_HOSTS` have multiple or all Elasticsearch nodes configured so that they can fail over.

### Update Elasticsearch cluster

Updating the Elasticsearch cluster happens one node after next. Before you star, ensure the following:

* You have three Elasticsearch nodes.
* Before proceeding with the next node it is strongly recommended to validate a new master has been elected.
* The remaining nodes have enough disk space to take over the shards from the node to be upgraded.

Follow these steps to update your Elasticsearch cluster:

1. On the Elasticsearch node you would like to update, navigate into your ELK solution directory:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Elasticsearch container you would like to update:

   ```bash
   docker stop elasticsearch1
   ```

3. Start a new Elasticsearch node (in this example, Elasticsearch Node1), which creates a new container based on the version you configured in your `.env` file:

    ```bash
    docker compose --env-file .env -f elasticsearch/docker compose.es01.yml up -d
    ```

Repeat these steps on the remaining Eleasticsearch nodes, but ensure a new Elasticsearch master has been elected.

### Update Kibana

To update Kibana, perform the following steps after adjusting the `ELASTIC_VERSION` accordingly.

1. On the Kibana node you would like to update, navigate to the directory where you installed Operational Insights:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Kibana container:

    ```bash
    docker stop kibana
    ```

3. Start a new Kibana container with the configured `ELASTIC_VERSION` in your `.env` file:

    ```bash
    docker compose --env-file .env -f kibana/docker compose.kibana.yml up -d
    ````

### Update Logstash

Follow these steps to update Logstash:

1. On the Logstash node you would like to update, navigate to the directory where you installed Operational Insights:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Logstash container:

    ```bash
    docker stop logstash
    ```

3. Start a new Logstash with the configured `ELASTIC_VERSION` in your `.env` file:

    ```bash
    docker compose up -d
    ```

{{< alert title="Note" >}}You must perform these steps on all Logstash nodes. {{< /alert >}}

### Update Filebeat

Follow these steps to update Filebeat:

1. On the Filebeat node you would like to update, navigate to the directory where you installed Operational Insights:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop existing filebeat container

    ```bash
    docker stop filebeat
    ````

3. Start a new Filebeat with the configured ELASTIC_VERSION in your .env file

    ```bash
    docker compose --env-file .env -f filebeat/docker compose.filebeat.yml up -d
    ```

{{< alert title="Note" >}}You must perform these steps on all Filebeat nodes. {{< /alert >}}
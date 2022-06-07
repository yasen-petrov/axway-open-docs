---
title: Get started with API Gateway in containers
linkTitle: Get started
weight: 20
date: 2019-09-18
description: Understand the differences between container deployments (API Gateway 7.6.0 and later) and classic deployments (API Gateway 7.5.3 and earlier) and get answers to frequently asked questions (FAQ) about deploying API Gateway in containers.
---

## How to deploy API Gateway in Docker containers

In API Gateway version 7.6.0 and later, you can deploy API Gateway in Docker containers and run it in externally managed topology (EMT) mode. This involves creating Docker images for the Admin Node Manager and API Gateway, and starting Docker containers from those images.

## How container deployments differ from classic deployments

There are significant differences when deploying in containers and running in EMT mode, compared to deploying and running in non-containerized classic mode.

|Container deployment|Classic deployment|
|---------|------|
| Docker manages API Gateway topology.         | Admin Node Manager manages API Gateway topology.   |
| An API Gateway domain comprises one Admin Node Manager and one or more API Gateways.  | An API Gateway domain comprises an Admin Node Manager, one or more Node Managers, and one or more API Gateways.  |
|API Gateway configuration (`.fed` file) is baked into the API Gateway Docker image. To deploy changes in API Gateway  configuration you must export a `.fed` file from Policy Studio, rebuild the API Gateway Docker image, and restart the API Gateway Docker container.|   To deploy changes in API Gateway configuration you can deploy directly from Policy Studio. |  
|You can easily scale a domain up or down by starting or stopping Docker containers.     | You cannot easily scale a domain up or down. The Admin Node Manager manages starting and stopping API Gateways. |

{{< alert title="Note" color="primary" >}}In a development environment, when you deploy in containers, you can use the `EMT_DEPLOYMENT_ENABLED` environment variable when starting an API Gateway container to enable deploying changes directly from Policy Studio to the running container. {{< /alert >}}

## Frequently asked questions about container deployments

### How do you deploy changes in configuration from Policy Studio

In a development environment only, you can set the `EMT_DEPLOYMENT_ENABLED` environment variable to `true` when starting an API Gateway Docker container. This enables you to deploy changes directly from Policy Studio to the container and to create Policy Studio projects from the running API Gateway container.

In a production environment, API Gateway configuration (`.fed` file) is baked into the API Gateway Docker image. To deploy changes in API Gateway configuration you must export a `.fed` file from Policy Studio, rebuild the API Gateway Docker image, and restart the API Gateway Docker container.

For more information, see [Development and deployment with API Gateway containers](/docs/apim_installation/apigw_containers/container_development).

### How do you promote configuration through environments

The promotion flow in a container deployment is very similar to a classic deployment, however, because API Gateway configuration (`.fed` file) is baked into the API Gateway Docker image, you must export the policy package ( `.pol` file) and environment package (.`env` file) from Policy Studio, rebuild the API Gateway Docker image, and restart the API Gateway Docker container, at each stage in the promotion flow.

For more information, see [Development and deployment with API Gateway containers](/docs/apim_installation/apigw_containers/container_development).

### How do you promote APIs through environments

In a container deployment, you can promote APIs registered in API Manager in the same way as in a classic environment. See
[Promote managed APIs between environments](/docs/apim_administration/apimgr_admin/api_mgmt_promote/).

### What API Gateway Manager functionality is not available

In a container deployment, you cannot perform the following in API Gateway Manager:

* You cannot create or delete API Gateway groups and instances
* You cannot start or stop API Gateways
* You cannot deploy configuration packages to a group of API Gateway instances

For more information, see [Operate and monitor API Gateway containers](/docs/apim_installation/apigw_containers/container_operations).

### How do you persist logs

To persist API Gateway logs to a directory on your host machine, you can use the `docker run -v` option to mount volumes for logs when running the API Gateway Docker container. For an example, see [Mount volumes to persist logs outside the API Gateway container](/docs/apim_installation/apigw_containers/docker_script_gwimage/#mount-volumes-to-persist-logs-outside-the-api-gateway-container).

For more information on using volumes to persist data, see the [Use volumes](https://docs.docker.com/storage/volumes/) Docker documentation.

### How do you customize audit logs

The audit log directory and filename can be customized by utilizing the following environment variables:

* **APIGW_AUDIT_LOG_DIR** - The directory into which audit logs are placed. Defaults to `VINSTDIR/logs`. `VINSTDIR` (the instance directory) is the location of the running server instance. For an API Gateway, this is the location where the server runs from. It is usually located in the `/groups/group-id/instance-id` directory, for classic mode, or the `/groups/emt-group/emt-service` directory for EMT. For a Node Manager, this the location where the product has been installed. It is usually located in the `/apigateway` directory. The value of this environment variable can itself be a selector, so as to facilitate the use of Java system properties and other environment variables as part of the name of the audit log directory.
* **APIGW_AUDIT_LOG_NAME** - The name of the audit log file. Defaults to `audit.log`. The value of this environment variable can itself be a selector, so as to facilitate the use of Java system properties and other environment variables as part of the name of the audit log.

These environment variables can be specified in the `docker run` command for the API Gateway instance as follows:

{{< alert title="Note" >}}`${environment.GROUPNAME}` and `${environment.INSTANCENAME}` are made available to the environment during API Gateway process startup. They are used here for illustrative purposes.{{< /alert >}}

```
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events \
           -e EMT_DEPLOYMENT_ENABLED=true -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e APIGW_AUDIT_LOG_DIR=\${environment.VINSTDIR}/logs/\${environment.GROUPNAME}
           -e APIGW_AUDIT_LOG_NAME=\${environment.INSTANCENAME}_audit.log \
           -e ACCEPT_GENERAL_CONDITIONS=yes \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false \
           -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-defaultgroup
```

If there is a requirement to persist the audit logs to a directory on your host machine, a mounted volume can be employed when running the API Gateway Docker container. The following `docker run` command results in the audit logs being persisted to the `/tmp/audit` directory on your host machine.

{{< alert title="Note" >}}For a multi-instance/multi-group topology scenario, using `${environment.GROUPNAME}` and `${environment.INSTANCENAME}` guarantees that audit logs are persisted to separate 'group' directories, with a unique audit log name per instance.{{< /alert >}}

```
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events \
           -v /tmp/audit:/opt/Axway/apigateway/groups/emt-group/emt-service/logs \
           -e EMT_DEPLOYMENT_ENABLED=true -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e APIGW_AUDIT_LOG_DIR=\${environment.VINSTDIR}/logs/\${environment.GROUPNAME}
           -e APIGW_AUDIT_LOG_NAME=\${environment.INSTANCENAME}_audit.log \
           -e ACCEPT_GENERAL_CONDITIONS=yes \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false \
           -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-defaultgroup
```

An alternative method of customizing the audit log directory and filename is by using a JMV.XML file. Given the following JVM.XML sample, the subsequent `docker run` command utilizes a mounted volume to inject the file into the docker container so that it is available when the API Gateway process starts up.

{{< alert title="Note" >}}In this example, the JVM.XML file resides in the `/tmp/emt/configurationFiles/groups/emt-group/emt-service/conf` directory of the host machine.{{< /alert >}}

```
<ConfigurationFragment>
    <Environment name="APIGW_AUDIT_LOG_DIR" value="$VINSTDIR/logs/$GROUPNAME"/>
    <Environment name="APIGW_AUDIT_LOG_NAME" value="$INSTANCENAME-audit.log"/>
</ConfigurationFragment>
```

```
docker run -d --name=apimgr --network=api-gateway-domain \
           -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events \
           -v /tmp/emt/configurationFiles:/merge/apigateway \
           -e EMT_DEPLOYMENT_ENABLED=true -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 \
           -e ACCEPT_GENERAL_CONDITIONS=yes \
           -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false \
           -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd \
           api-gateway-defaultgroup
```

### How do you customize logs in a multi-node environment

You can customize the logs produced by API Gateway in Policy Studio by way of the following environment variables:

* HOSTNAME - `${environment.HOSTNAME}`
* GROUPNAME- `${environment.GROUPNAME}`
* INSTANCENAME- `${environment.INSTANCENAME}`

These variables allow you to make the configuration generic, so that you can create unique log directories and filenames across a multi-node EMT environment

To customize logging for a multi-node environment, perform the following steps:

1. In Policy Studio, create a project [from a .fed file](/docs/apim_policydev/apigw_poldev/gs_project/#new-project-from-a-fed-file), pointing it to an existing API Gateway FED. Or, alternatively, create a project [from an existing configuration](/docs/apim_policydev/apigw_poldev/gs_project/#new-project-from-existing-configuration) pointing to an existing directory that contains an XML or YAML configuration.
2. Click **Server Settings > Logging**.
3. Click **Transaction Audit Log** and configure where you wish API Gateway to log transaction audit information:

   To log to a text file, select the **Text File** tab and configure the following:
   * Check **Enable logging to file**.
   * Set **File Name** to `transactionLog-${environment.HOSTNAME}`.
   * Set **Directory** to `logs/transaction-${environment.HOSTNAME}`.

   To log to an XML file, select the **XML File** tab  and configure the following:
   * Check **Enable logging to XML file**.
   * Set **File Name** to `transactionLog-${environment.HOSTNAME}`.
   * Set **Directory** to `logs/transaction-${environment.HOSTNAME}`.
4. Click **Transaction Access Log** and configure where you wish the API Gateway to log transaction access log information:
    * Check **Writing to Transaction Access Log**.
    * Set **File Name** to `access-${environment.HOSTNAME}`.
    * Set **Directory** to `logs/access-${environment.HOSTNAME}`.
5. Click **Transaction Event Log** and configure where you wish API Gateway to log transaction event log information:
    * Check **Writing to Transaction Event Log**.
    * Set **Write transaction event logs to directory** to `logs/events-${environment.HOSTNAME}`.
6. click **Traffic Monitor** and configure where you wish API Gateway to log traffic information:
    * Set **Transaction Directory** to `conf/opsdb.d/opsdb-${environment.HOSTNAME}`.

You can rebake the updated API Gateway .fed file into an API Gateway Docker image, or you can use Docker volumes to update the configuration without having to rebake an image. For more information, see [Create an API Gateway with Docker volumes](/docs/apim_howto_guides/configuring_apigw_container/index.html).

### What license do you need to run

You must have an appropriate license to run API Gateway or API Manager in a Docker container. For more information, see [Set up Docker environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs).

### How do you upgrade from 7.5.3 to 7.7

To upgrade from API Gateway 7.5.3 (classic deployment) to API Gateway 7.7 (container deployment), you must first upgrade to a 7.7 classic deployment, and then migrate to a 7.7 container deployment.

For information on upgrading to 7.7, see the
[API Gateway Upgrade Guide](/docs/apim_installation/apigw_upgrade/)
, and for more details on migrating to a container deployment, see [Migrate to container deployment](/docs/apim_installation/apigw_containers/container_migration).

### How do you create API Gateway Docker images with customized configuration

When building your API Gateway or Admin Node Manager Docker images, you can specify a merge directory containing custom configuration, JAR files, and so on, to add to the Docker image. For detailed examples, see:

* [Create an ANM image using existing ANM fed and customized configuration](/docs/apim_installation/apigw_containers/docker_script_anmimage/#create-an-admin-node-manager-image-using-existing-fed-and-customized-configuration)
* [Create an API Gateway image using existing fed and customized configuration](/docs/apim_installation/apigw_containers/docker_script_gwimage/#create-an-api-gateway-image-using-existing-fed-and-customized-configuration)

### How do you manage API Gateway topology

In a classic deployment, topology is managed internally through an Admin Node Manager communicating with Node Managers in API Gateway nodes. In a container deployment, the topology is externally managed by a cluster manager, such as Docker Swarm or Kubernetes, and it manages the topology and communicates directly with each API Gateway.

This means that you cannot use the `managedomain` script or the API Gateway Manager web UI to manage your topology in a container deployment. Instead, you must use an orchestration tool. For more information on Docker Swarm, see the [Getting started with swarm mode](https://docs.docker.com/engine/swarm/swarm-tutorial/) Docker documentation, or for more information on Kubernetes go to the [Kubernetes website](https://kubernetes.io/).

### How do you run API Gateway administration tools or scripts

In a container deployment, you must connect to the running Admin Node Manager Docker container to run API Gateway administration tools, such as `kpsadmin`. For an example, see [Manage KPS with kpsadmin](/docs/apim_installation/apigw_containers/container_troubleshoot/#manage-kps-with-kpsadmin).

### How do you set up API Manager metrics

To set up API Manager metrics you must first create an Admin Node Manager Docker image with metrics processing enabled, and then run the Admin Node Manager and API Gateway Docker containers using the `docker run -v` option to mount a volume for the API Gateway events directory.

When starting the containers, you must also specify the connection details for the metrics database using environment variables. For an example, see [Create a metrics-enabled ANM image](/docs/apim_installation/apigw_containers/docker_script_anmimage/#create-a-metrics-enabled-admin-node-manager-image).

### How do you run API Gateway in Kubernetes

To run API Gateway in Kubernetes, you can use the [API Gateway Helm charts on GitHub](https://github.com/Axway/apigw-helm-charts).

These Helm charts are not intended as a recommended setup, but something that you can use as a starting point and customize for your own environment.

### How do you run API Gateway with a persisted volume

[Deploy API Gateway in containers](/docs/apim_installation/apigw_containers/) outlines how to create API Gateway images with fully baked configuration. Users may wish to make configuration updates in a more targeted manner, this is facilitated through the use of persisted volumes. The [Docker volumes with API Gateway](/docs/apim_howto_guides/configuring_apigw_container) section outlines the basics for how this approach can be implemented.

---
title: Deploy API Gateway using Helm
linkTitle: Deploy API Gateway using Helm
weight: 107
date: 2022-10-17
description: Follow this section to install API Gateway and API Manager using a Helm chart.
---

## Prerequisites

* Knowledge of, and preferably experience with using [Helm](https://helm.sh).
* You must have access to the Axway public Helm repository, `helm.repository.axway.com`, to download or directly access API Gateway public Helm charts.
* You must have access to a Kubernetes cluster, for example in AWS, Azure, Openshift, and so on. For details on possible cloud based architectures, see [API management reference architectures](/docs/apim-reference-architectures/).
* You must have a persistent storage with access mode of `RWO` and `RWX`, and which is exposed as a Kubernetes storage class, for example, OpenShift Container Storage, Azure Disks/Files, AWS EBS/EFS, nfs-client.

{{< alert title="Note" color="primary" >}}In the sample commands on this page, when not specifically mentioned, we assume the cluster is an Openshift cluster. In that environment, the openshift CLI command `oc` is used. For other Kubernetes clusters, replace `oc` with `kubectl`. The `helm` command is the same on all clusters. {{< /alert >}}

## Build Docker images

Follow this section to build your API Gateway Docker images.

1. Follow the prerequisites to set up your Docker environment and to set up API Gateway Docker scripts as detailed in [Set Up Docker Environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/#set-up-your-docker-environment).
2. Build a base image as detailed in [Create base Docker image](/docs/apim_installation/apigw_containers/docker_script_baseimage).
3. Build an Admin Node Manager (ANM) image as detailed in [Create an Admin Node Manager Docker image](/docs/apim_installation/apigw_containers/docker_script_anmimage/#create-an-admin-node-manager-docker-image).
4. Build the `Group1` API Gateway image as detailed in [Create an API Gateway Docker image](/docs/apim_installation/apigw_containers/docker_script_gwimage/#create-an-api-gateway-docker-image).

## Publish Docker images

After the Docker images have been built for the API Gateway, they must be tagged and pushed to a container registry which is accessible by the Kubernetes cluster. Follow this section to publish your images.

1. Log in to the container registry, for example, to the Openshift Container Registry (OCR).
2. Get the registry hostname of your container registry. For example, in openshift you might be able to retrieve the URL from its route using the following command:

    ```bash
    oc get routes -n openshift-image-registry default-route
    ```

3. Authenticate to the registry:

    ```bash
    docker login -u <username> -p $(oc whoami -t) $REGISTRY_URL
    ```

4. Tag the local ANM image. (This tag is later used in your customized `values.yaml` file).

    ```bash
    docker tag admin-node-manager $REGISTRY_URL/apigw/anm:1.0
    ```

5. Push the image to the image registry:

    ```bash
    docker push $REGISTRY_URL/apigw/anm:1.0
    ```

After the images have been pushed to the registry, they will then be available to be used with the Helm chart. This is done by referencing them in your customized `values.yaml` file.

## Deploy a Cassandra cluster

After you publish your API Gateway Docker images, you must deploy a Cassandra cluster to your project. Deploying a Cassandra cluster in containers is only recommended for development environments. In a production environment, you must configure Cassandra for high availability (HA) as detailed in [Configure a Cassandra HA cluster](/docs/cass_admin/cassandra_config/).

### Deploy Cassandra in a development environment

In a development environment, Cassandra can be run in containers and the cluster can be easily deployed using a public Helm chart. Follow this section to deploy Cassandra in a development environment.

1. In OpenShift, create a new project for the cassandra deployment:

    ```bash
    oc new-project cassandra
    ```

    Alternatively, in a non-openshift cluster, run the equivalent command:

    ```bash
    kubectl create namespace cassandra
    ```

2. Add the `bitnami` Helm repo to the project:

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    ```

3. Using the [`bitnami/cassandra`](https://github.com/bitnami/charts/blob/main/bitnami/cassandra) chart, install Cassandra by running customized version of this sample command:

    ```bash
    helm install cassandra bitnami/cassandra \
         --set podSecurityContext.enabled="false" \
         --set containerSecurityContext.enabled="false" \
         --set dbUser.user=cassandra \
         --set dbUser.password=REDACTED \
         --set resources.requests.memory=8Gi \
         --set replicaCount=3 \
         --set image.tag="3.11.13" \
         --set jvm.maxHeapSize=1024M \
         --set jvm.newHeapSize=1024M \
         -n cassandra
    ```

## Deploy a MySQL database

If you are using metrics for analytics, follow this section to deploy a MySQL database for your metrics.

1. Create a project for the MySQL instance:

    ```bash
    oc new-project metrics
    ```

    Alternatively, in a non-openshift cluster, run the equivalent command:

    ```bash
    kubectl create namespace metrics
    ```

2. Prepare a `configmap` with the SQL file for the schema init:

    ```bash
    echo 'CREATE DATABASE metrics; USE metrics;' | cat - mysql-analytics.sql > mysql-analytics-cm.sql
    kubectl create configmap mysql-metrics --from-file=mysql-analytics-cm.sql -n metrics
    ```

    The `mysql-analytics.sql` file is found in the API Gateway Docker scripts, in the `/quickstart` directory. For more information on the use of the this file, see [Start the metrics database](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/#start-the-metrics-database).

3. Install the Helm chart using [bitnami/mysql](https://github.com/bitnami/charts/tree/main/bitnami/mysql):

    ```bash
    helm install mysql bitnami/mysql \
         --set primary.podSecurityContext.enabled="false" \
         --set primary.containerSecurityContext.enabled="false" \
         --set secondary.podSecurityContext.enabled="false" \
         --set secondary.containerSecurityContext.enabled="false" \
         --set auth.rootPassword=password \
         --set initdbScriptsConfigMap=mysql-metrics \
         --set image.tag=8.0 \
         --set primary.persistence.storageClass="nfs-client" \
         -n metrics
    ```

## Use Helm to install API Gateway

Download the API Gateway Helm chart from `repository.axway.com`. The chart is packaged as a `tar.gz` file, and it can be used directly as a `tar.gz` file, but you must unnzip it to access the `values.yaml` file.

There are two ways to customize your deployment:

* Directly edit the `values.yaml` file and modify it with your own customized values.
* Create a customized `values` file, for example, `myvalues.yaml`, and make your customizations. This file should contain only the sections of the `values.yaml` file that you wish to override. Any values not present in the customized file will be picked up in the original `values.yaml` file.

The following is an example of a customized `myvalues.yaml` file using the ANM image created earlier:

```bash
nameOverride: gateway
global:
  domainName: myexample.com
  defaultRegistry: mydocker-registry-url.myexample.com/apim
  imagePullPolicy: Always
  storage:
    storageClassName: nfs-client
    volumes:
      - enabled: true
        name: events
        usedBy:
          - aga
          - anm
          - apimgr
          - traffic
        accessModes:
          - ReadWriteMany
        capacity: 1Mi
      - name: opentraffic
        enabled: true
        accessModes:
          - ReadWriteMany
        capacity: 1Mi
        persistentVolume:
          csiDriver: ""
          volumeHandle: ""
          reclaimPolicy: Delete
      - name: payloads
        enabled: true
        accessModes:
          - ReadWriteMany
        capacity: 1Mi
        persistentVolume:
          csiDriver: ""
          volumeHandle: ""
          reclaimPolicy: Delete
  database:
    host: mysql.metrics.svc.cluster.local
    metrics:
      enabled: true
      username: "apim"
      password: "redacted"
  cassandra:
    enabled: true
    hosts:
      - variable: CASS_HOST
        hostname: cassandra.cassandra.svc.cluster.local
    username: cassandra
    password: cassandra
    keyspace: ks
    tkeyspace: tks

anm:
  image:
    repository: "anm"
    tag: "1.0"
  route:
    enabled: false
  ingress:
    enabled: true
    className: nginx
    annotations:
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
    hosts:
      - host: anm.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - anm.myexample.com

apimgr:
  image:
    repository: "apigw"
    tag: "mytag-01"
  resources:
    limits:
      memory: "2Gi"
      cpu: 2
    requests:
      memory: "0.5Gi"
      cpu: 0.5
  securityContext:
    runAsNonRoot: false
  route:
    enabled: false
  ingress:
    enabled: true
    className: nginx
    annotations:
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
    hosts:
      - host: apimgr.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - apimgr.myexample.com

apitraffic:
  image:
    repository: "apigw"
    tag: "mytag-01"
  ingress:
    enabled: true
    className: nginx
    annotations:
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
    hosts:
      - host: apitraffic.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - apitraffic.myexample.com
  extraVolumeMounts:
    - name: events
      mountPath: /opt/Axway/apigateway/events
    - name: opentraffic
      mountPath: /opt/Axway/apigateway/logs/opentraffic
    - name: payloads
      mountPath: /opt/Axway/apigateway/logs/payloads
  extraVolumes:
    - persistentVolumeClaim:
        claimName: events
      name: events
    - persistentVolumeClaim:
       claimName: opentraffic
      name: opentraffic
    - persistentVolumeClaim:
       claimName: payloads
      name: payloads

aga:
  enabled: true
  route:
    enabled: true
  image:
    repository: "aga"
    tag: "mytag-01"
  ingress:
    enabled: true
    className: nginx
    annotations:
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
    hosts:
      - host: analytics.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - analytics.myexample.com
```

Follow these steps to install API Gateway using your customized `myvalues.yaml` file.

1. Using the `kubectl` or `oc` commands, create a namespace for your API Gateway Helm chart, for example:

    ```bash
    oc create namespace mynamespace
    ```
2. Use your customized file to install the chart. The following command will pick up all values from the original `values.yaml` and override them with any specific values from `myvalues.yaml`.

    ```bash
    helm install -f myvalues.yaml apim -n mynamespace
    ```

    If just using a modified `values.yaml` file, then run:

    ```bash
    helm install -f values.yaml apim -n mynamespace
    ```

3. After the installation is finished, check the deployment, for example:

    ```bash
    kubectl get pods -n mynamespace
    ```

    Result:

    ```none
    NAME                                       READY     STATUS    RESTARTS   AGE
    apim-gateway-aga-866946bb58-vsqlx          1/1       Running   0          72s
    apim-gateway-anm-58b5644777-jbzfl          1/1       Running   0          72s
    apim-gateway-apimgr-654fcd4744-xqxcw       0/1       Running   0          72s
    apim-gateway-apitraffic-6fc46d6788-s4mk9   0/1       Running   0          72s
    ```

4. After all the pods are up and running, you should be able to reach your Admin Node Manager, your API Gateway, the API Manager, and the Analytics UI, if enabled. To check these, get the ingresses and use them to find the URL for each of the interfaces:

    ```bash
    kubectl get ing -n mynamespace
    ```

    Result:

    ```none
    NAME                      CLASS     HOSTS                      ADDRESS   PORTS     AGE
    apim-gateway-aga          nginx     analytics.myexample.com              80, 443   13s
    apim-gateway-anm          nginx     anm.myexample.com                    80, 443   13s
    apim-gateway-apimgr       nginx     apimgr.myexample.com                 80, 443   13s
    apim-gateway-apitraffic   nginx     apitraffic.myexample.com             80, 443   13s
    ```

    Then, browse to one of the services, for example, <https://anm.my.example.com> to reach the Admin Node Manager.

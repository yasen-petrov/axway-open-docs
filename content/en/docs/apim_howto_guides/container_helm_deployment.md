---
title: Deploy API Gateway using Helm
linkTitle: Deploy API Gateway using Helm
weight: 107
date: 2022-10-27
description: Follow this section to install API Gateway and API Manager using a Helm chart.
---

## Prerequisites

* Knowledge of, and preferably experience with using [Helm](https://helm.sh).
* Download the API Gateway helm charts from [Axway repository](http://respository.axway.com/)
* You must have access to a Kubernetes cluster, for example in AWS, Azure, Openshift, and so on. For details on possible cloud based architectures, see [API management reference architectures](/docs/apim-reference-architectures/).
* You must have a persistent storage with access mode of `RWO` and `RWX`, and which is exposed as a Kubernetes storage class, for example, OpenShift Container Storage, Azure Disks/Files, AWS EBS/EFS, nfs-client.

{{< alert title="Note" color="primary" >}}In the sample commands on this page, when not specifically mentioned, we assume the cluster is an Openshift cluster. In that environment, the openshift CLI command `oc` is used. For other Kubernetes clusters, replace `oc` with `kubectl`. The `helm` command is the same on all clusters. {{< /alert >}}

## Download Docker images

Download the latest Axway docker images from [Axway repository](http://respository.axway.com/) or alternatively, follow section [Generate custom API Management Docker images](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/) to generate a custom Docker image.

## Publish Docker images

The Docker images must be tagged and pushed to a container registry which is accessible by the Kubernetes cluster. Follow this section to publish your images.

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

    The `mysql-analytics.sql` file is found in the API Gateway Docker scripts, in the `/quickstart` directory. For more information on the use of the this file, see [Start the metrics database](/docs/apim_installation/apigw_containers/docker_scripts_prereqs#start-the-metrics-database).

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
      - name: anm-external-config
        enabled: true
        usedBy:
          - anm
        accessModes:
          - ReadWriteMany
        persistentVolume:
          csiDriver: ""
          volumeHandle: ""
          reclaimPolicy: Delete
        capacity: 5Mi
      - name: gw-external-config
        enabled: true
        usedBy:
          - apimgr
          - apitraffic
        accessModes:
          - ReadWriteMany
        persistentVolume:
          csiDriver: ""
          volumeHandle: ""
          reclaimPolicy: Delete
        capacity: 5Mi
      - name: aga-external-config
        enabled: true
        usedBy:
          - aga
        accessModes:
          - ReadWriteMany
        persistentVolume:
          csiDriver: ""
          volumeHandle: ""
          reclaimPolicy: Delete
        capacity: 5Mi
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
#  domainkeypassphrase:
#    passphrase: "redacted"

anm:
  image:
    repository: "anm"
    tag: "1.0"
  resources:
    limits:
      memory: "2048Mi"
      cpu: "1000m"
    requests:
      memory: "1Gi"
      cpu: "250m"
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
      - host: anm.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - anm.myexample.com
  service:
    port: 8091
  extraVolumeMounts:
    - name: anm-external-config
      mountPath: /merge
  extraVolumes:
    - persistentVolumeClaim:
        claimName: anm-external-config
      name: anm-external-config

apimgr:
#  groupId: 
#    "ApiMgrGroup"
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
  service:
    port: 8075
  extraVolumeMounts:
    - name: gw-external-config
      mountPath: /merge
  extraVolumes:
    - persistentVolumeClaim:
        claimName: gw-external-config
      name: gw-external-config
  license:
    license.lic: |
#      redacted

apitraffic:
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
  oauth:
    route:
      enabled: false
  route:
    enabled: false
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
  service:
    port: 8065
  extraVolumeMounts:
    - name: events
      mountPath: /opt/Axway/apigateway/events
    - name: gw-external-config
      mountPath: /merge
  extraVolumes:
    - persistentVolumeClaim:
        claimName: events
      name: events
    - persistentVolumeClaim:
        claimName: gw-external-config
      name: gw-external-config
  license:
    license.lic: |
#      redacted

aga:
  image:
    repository: "aga"
    tag: "mytag-01"
  enabled: true
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
      - host: anm.myexample.com
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls:
      - hosts:
          - anm.myexample.com
  service:
    port: 8040
  resources:
    limits:
      memory: "2048Mi"
      cpu: "1000m"
    requests:
      memory: "1Gi"
      cpu: "250m"
  extraVolumeMounts:
    - name: aga-external-config
      mountPath: /merge
  extraVolumes:
    - persistentVolumeClaim:
        claimName: aga-external-config
      name: aga-external-config
  license:
    license.lic: |
#      redacted
```

### Add a license configuration

Product license configuration can be added to each component individually by way of a Helm ConfigMap. The `license.lic` license file must be configured within the `License` ConfigMap.

### Add a group ID configuration

The group ID can be set on the API Manager or API Traffic component. For example:

```bash
apimgr:
  groupId: 
    "APIMgrGroup"

apitraffic:
  groupId: 
    "APITrafficGroup"
```

### Add a domain certificate configuration

Domain certificates can be loaded to the environment by way of the persistent volume mount point for `apigateway` configuration files. For more information, see [Mount component configuration](/docs/apim_howto_guides/container_helm_deployment/#mount-component-configuration) section.

If the domain certificates were generated with a passphrase, that passphrase must be configured within the global `domainkeypassphrase`. For example:

```bash
global:
  domainkeypassphrase:
    passphrase: "redacted"
```

### Configure Entity Store Passphrase

Any component which uses an Entity Store with a passphrase can configure the passphrase in the `values` override file for that specific component. For example:

```bash
anm:
  espassphrase: "redacted"
```

### Install API Gateway using your customized YAML file

Follow these steps to install API Gateway using your customized `myvalues.yaml` file.

1. Using the `kubectl` or `oc` commands, create a namespace for your API Gateway helm chart, for example:

    ```bash
    oc create namespace mynamespace
    ```
2. Use your customized file to install the chart. The following command will pick up all values from the original `values.yaml` and override them with any specific values from `myvalues.yaml`.

    ```bash
    helm install -f myvalues.yaml apim -n mynamespace <repository.axway.com>
    ```

    If just using a modified `values.yaml` file, then run:

    ```bash
    helm install -f values.yaml apim -n mynamespace <repository.axway.com>
    ```

    {{< alert title="Note" >}}Please refer to repository.axway.com for the most recent version{{< /alert >}}

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

### Mount a component configuration

Components can be configured to mount custom configurations, which can then be made available to the pod at runtime. Each component has a configured persistent volume to facilitate this. The following table details the expected mount locations for each configuration type. See [API Gateway configuration files](/docs/apim_reference/config_files_reference/) for a full list of configurable files.

| Configuration Type | Pod mount point | Description |
| ------------------ | --------------- | ----------- |
| Entity Store (.fed) | /merge/fed | For a policy configuration stored as a deployment package (.fed file).|
| Entity Store (.yaml) |/merge/yaml | For YAML based API Gateway policy configuration stored either as a deployment package (.tar.gz file) or as a directory. This option is applicable for API Gateway only.|
| Mandatory Files Verification | /merge/mandatoryFiles | For the verification of mandatory configuration files. |
| Apigateway Configuration | /merge/ | For all other API Gateway or Admin Node Manager configurations, for example, jvm.xml, envSettings.props, and so on.|
| Analytics Configuration | /merge/ | For all Analytics configurations, for example, jvm.xml, envSettings.props, and so on.|

#### The structure of configuration directories

`apigateway` and `analytics` configuration files must be in the same directory structure which mirrors the `/opt/Axway/apigateway` or `/opt/Axway/analytics` installation directories. For example:

```bash
apigateway
├── groups
│   ├── certs
│   │   ├── domaincert.pem
│   │   └── private
│   │       └── domainkey.pem
│   └── emt-group
│       └── emt-service
│           └── conf
│               ├── envSettings.props
│               └── jvm.xml
└── system
    └── conf
        └── log4j2.yaml
```

Configuration files can be copied to the persistent volumes using tools, such as `kubectl` and `scp`. For example:

```bash
# Copying a fed file to the APIMgr component
kubectl cp apimgr.fed <apimgr pod id>:/merge/fed

# Copying an apigateway configuration directory to the APIMgr component
kubectl cp apigateway <apimgr pod id>:/merge/
```

Once the external configuration has been successfully copied to the relevant persistent volume, the deployment can be reloaded in order for the new configuration to be adopted to the relevant component. For example:

```bash
# Reload the API Manager deployment
kubectl rollout restart deployment apigw-gateway-apimgr -n <namespace>
```

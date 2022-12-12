---
title: Configure a setup for production environments
linkTitle: Production setup
weight: 40
no_list: false
date: 2022-07-20
description: 
---

Advanced configuration topics that are required to run Operational Insights in a production environment

{{< alert title="Note" >}}
The ELK stack is entirely owned and operated by the user.

This section is focused on the integration layer between the API Gateway and the ELK stack, facilitated by the API Builder image. As such, default security settings are set, but they must reviewed and enhanced based on the [Elasticsearch Security](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-cluster.html) documentation.
{{< /alert >}}

## Prerequisite

* You have familiarized yourself with Operational Insights by using its [basic setup](/docs/operational_insights/basic_setup/).

## Securing a kubernetes cluster

For a general overview of how to secure a production cluster, see the [Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/) documentation.

Following we describe three specific areas of security for your Kubernetes cluster: network policies, secrets, and encryption in transit.

### Kubernetes networking

In kubernetes, containers run inside pods. A typical pod will only run one container, but it is also common to run more than one container in a pod if the other containers are helper or sidecar containers.

In a kubernetes cluster, each pod gets its own IP address, which is unique throughout the cluster. Each container within a pod uses the same IP address, there is no isolation between the pods, meaning that all pods within a cluster can communicate with each other without Network Address Translation. This applies to pods running in different namespaces or on different nodes.

If you want to restrict access from some pods to others, especially in a multi-tenancy cluster, you must configure network policies.

#### Network policies

Network policies are similar to firewalls, they allow you to control traffic flow between pods at the IP address or port level. This allows control of traffic coming from other pods, other namespaces, or from the outside world. This incoming traffic (ingress), which is disabled by default, can be opened and controlled by way of a network policy. Outgoing traffic from the pods to the outside world (egress), which is enabled by default, can also be controlled as part of a network policy.

In summary, network policies work by:

* Controlling access from pod to pod.
* Granting or denying pods access from or to a namespace.
* Using IP CIDR blocks to restrict access to pods.

#### Container Network Interface (CNI) plugins

Network policies cannot be enforced without a CNI plugin. If running kubernetes on a managed service such as Azure Kubernetes Service (AKS) you can avail of the built-in Azure CNI. Otherwise, you can install a third-party plugin such as Calico or Weaveworks.

For more information on kubernetes networking, including sample network policies, see the [Kubernetes](https://kubernetes.io/docs/concepts/services-networking/network-policies)  documentation, or the [Calico](https://projectcalico.docs.tigera.io/about/about-network-policy) documentation.

#### Default networking in helm charts

By default, Operational Insights helm chart does not use network policies but it directly enables ingress traffic on the elasticsearch and kibana pods to allow inbound communication to take place. For the other pods, ingress is either disabled or not configure, which means that no inbound traffic is allowed into these pods.

### Kubernetes secrets

In Operational Insights, passwords such as those for API Manager, kibana, logstash, and Elasticsearch are stored in clear text in environment variables. This makes them easily accessible to anyone who has access to the kubernetes cluster.

Kubernetes secrets provide a further layer of security. They are stored as key-value pairs and can be created outside of the application code and referenced from within the code. Secrets are base-64 encoded and so are still accessible to anyone who has the credentials to query elements in the cluster.

In a production environment we recommend that you store your passwords using kubernetes secrets that are backed by a secrets manager such as Hashicorp Vault, AWS Secret Manager, or Azure Vault. Access to secrets should also be restricted using RBAC (role-based access control) rules.

For more information on kubernetes secrets, see the [kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/) documentation.

### Elasticsearch encryption in transit

By default, all data sent between nodes in the Elasticsearch cluster is sent in clear text. This might be acceptable if the cluster itself is sufficiently secured. However, in a production environment, we recommend that traffic between nodes is encrypted. To encrypt your traffic, locate the `elasticsearch.yml` file, then follow the instructions on [Encrypt internode communications with TLS](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#encrypt-internode-communication).

If your cluster is running in a managed service, such as Azure Kubernetes Service (AKS), then you will have built-in security options, such as enabling TLS, which come as part of the service.

For more details on encryption in transit, see the following documentation:

* [Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#security-basic-setup)
* [Azure](https://docs.microsoft.com/en-us/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)

## Consult long-term analytics

Operational Insights component supports long-term analytics capabilities out of the box in addition to relatively short-term operations and the corresponding real-time dashboard. For this purpose, the highly granular raw data received from API Gateway instances is transformed into entity-centric indices that are pre-aggregated and require only a fraction of the necessary disk space. Note that this reduces the ability to analyze data down to the minute.

To transform the raw data, Operational Insights delivers and automatically installs a ready-made transformation job and provides corresponding dashboards. The transformation works with a delay of three hours. This means that real-time data will show in the Quarterly and Yearly dashboards only after this time. This delay allows you to suspend ingesting data for a maximum of three hours, so the transformation job will not lose data.

Long-term analytics data will only be available after approximately one hour of the start of the transformation. Until then, the corresponding dashboards will show errors. For example,

```none
The field "name-of-a-field" associated with this object no longer exists. Please use another field. Please wait at least 1 hour for the data to be prepared accordingly or create the transformation job manually executing the following command in the APIBuilder4Elastic container:

wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/transform/apigw-traffic-summary
```

## Where to go next

Follow the specific sections to configure your production environment using Helm or Docker Compose.
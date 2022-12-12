---
title:  Operational Insights FAQ
linkTitle: FAQ
weight: 400
date: 2022-06-20
description: Frequently asked questions and answers when using the Operational Insights component.
---

## Do I need an API Builder subscription?

No. The delivered API builder Docker image includes the community version and is supported as part of this project.

## Do I need an Elastic subscription?

The component can currently be operated in production without a subscription. If you need official Elastic support to run your Elastic Stack, or additional features like alerts or machine learning for operation, then you need a subscription to Elastic. You can find more information at [Elastic pricing](https://www.elastic.co/pricing/).

## Do I need a specific API Gateway version to use this solution?

Any API Gateway version that supports the [Open Traffic Event Log](/docs/apim_reference/monitor_traffic_events_metrics/#open-traffic-event-log-settings) is supported. The component has been tested with the API Management version 7.7 [July 2020](https://axway-open-docs.netlify.app/docs/apim_relnotes/20200730_apimgr_relnotes/) update.

## Will indexed data be deleted automatically?

Yes. Each index created in Elasticsearch is assigned an Index Lifecycle Policy ([ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)). This controls how long data is retained for each data type. For more information, see [Configure the retention period](/docs/op_insights/production_setup/op_insights_setup_prod_docker/#configure-the-retention-period).

## Can I use my own existing Elasticsearch cluster?

Yes. You can use your own Elasticsearch cluster, as long as it is a `> 7.10.x` version with X-Pack features enabled and supported. For instance, AWS-Elasticsearch service does not provide X-Pack and, therefore is not supported. Additionally, a version higher than `7.10` is required to run the transformation job correctly. For more information about missing buckets, see issue [#59591](https://github.com/elastic/elasticsearch/pull/59591) on the Elasticsearch GitHub project.

## Does the component supports high availability?

Yes. All components can be deployed in a way to support HA for the entire platform.

## Can I use the Filebeat version shipped with the API Gateway?

No, as it is not a `7.x` version and it is not tested with this component yet.

## Can I run Filebeat as a native process?

Yes. You can run Filebeat natively instead of a Docker container as long as you are using the `filebeat.yml` with the correct configuration. However, the component has been tested with Filebeat 7.12.

## What happens if Logstash is down for a while?

If Logstash is not running, Filebeat cannot send events to Logstash. However, Filebeat remembers the position of the last sent events on each files and resumes at that position when Logstash is available again. You must ensure that the files available are large enough. For instance, the OpenTraffic Event Logs are configured by default to 1GB, which is sufficient for around 30 minutes when having 300 TPS. You should increase the disk space if this is not sufficient.

## What happens if Filebeat is down for a while?

Filebeat stores the current position in a file (registry), storing which events have already been passed on. If Filebeat is shut down cleanly and then restarted, it will pick up where it left off. However, it is still possible that some events will be lost.

## Can I run multiple Logstash instances?

Yes. Logstash is stateless (besides what is stored is Memcache), hence, you can run as many Logstash instances as you like or need. In that case, you must provide multiple Logstash instances using the parameter `LOGSTASH_HOSTS` to the Filebeat instances so that they can load balance the events.

## Can I run multiple API Builder instances?

Yes. Ensure to provide the same configuration for each API Builder Docker Container. It is recommended to run the API Builder process alongside Logstash and Memcache, as these components work closely together. Low latency between the Logstash and Memcache containers is crucial.

## Can I customize or change this component to my needs?

It is not recommended that you change the component just by changing some of the files, as they are very likely be overwritten with the next version. For more information, see [About components update](/docs/operational_insights/update_operational_insights/op_insights_update_docker/#about-components-update) for more details.

## Are traces stored in Elasticsearch?

Yes. The trace messages you see in the Traffic Monitor for an API request are retrieved from the Elasticsearch cluster. However, running an API Gateway in DEBUG mode, handling millions of transactions a day, will significantly increase the disk space requirements in Elasticsearch and in the number of events to be processed. When enabling DEBUG, monitor your Elasticsearch stack to ensure event processing stays real-time.

## Can you encrypt passwords added to the env file?

No, but this file is used by Docker Compose only. If you would like to avoid storing sensitive data, the recommended approach it to deploy this component in a Kubernetes environment and store passwords in the [Vault](https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-security-concerns). This way, sensitive data is injected into the containers from a secure source.

## Why do Kibana dashboards show different volumes than the Traffic Monitor?

If the Traffic Monitor dashboard shows significantly more transactions than the Kibana dashboards, for example, for the last 10 minutes, this is likely due to the ingestion rate not being high enough. In other words, the ELK stack cannot process the data fast enough. If the behavior does not recover, you might need to add more Logstash or Elasticsearch nodes.

## Can I increase the disk volume on an Elasticsearch machine?

Yes. All indices are configured to have one replica and therefore, it is safe to stop a machine, increase the disk space, then restart it. Ensure the cluster state is `Green` before stopping an instance. The increased volume will be automatically detected by Elasticsearch and used to assign more shards to it. It is recommended to assign the same disk space to all Elasticsearch cluster nodes.

## Can I use AWS Elasticsearch service?

No. The component does not support AWS Elasticsearch Service because some important features that are part of X-Pack are not supported by AWS. For example, index lifecycle management (ILM).

## Filebeat is reporting errors

When Filebeat reports errors like the following, it might be that the registry is corrupted. When this happens, Filebeat basically stops for a second to send events, which might cause issues to stay real-time when running very high volume.

```bash
Harvester could not be started on new file: /var/log/opentraffic/group-2_instance-1_traffic.log, Err: error setting up harvester: Harvester setup failed. Unexpected file opening error: file info is not identical with opened file. Aborting harvesting and retrying file later again
```

## Which protocols are supported?

HTTP, JMS, and file transfer are the protocols supported. This means that whenever you run a query via the API Gateway Traffic Monitor for one of these protocols, the data will come from Elasticsearch, otherwise it will continue to come from the OpsSDB.

## Why do Administrators only see JMS Traffic?

JMS requests are not controlled by the API Manager, therefore there is no association with an organization and therefore, the result cannot be restricted accordingly. If you wish, you can disable the complete user authorization by setting the `enableUserAuthorization` parameter to `false`. For more information, see, [Customize user authorization](docs/operational_insights/additional_features/op_insights_custom_user_authz), or you can use the `UNRESTRICTED_PERMISSIONS` parameter to configure which users should see the entire traffic.

## Is EMT mode supported?

Yes. The component can be used when the API Management platform is deployed in a Docker orchestration platform in externally managed topology ([EMT](/docs/apim_installation/apigw_containers/container_intro/)) mode. This way, it is possible, for instance, to see traffic from containers (PODs) that have already been removed from the Traffic Monitor.

## Can I disable the OpsDB for traffic monitoring?

No. The settings can be found in Policy Studio **Server Settings > Monitoring > Traffic Monitor**. You must ensure that Traffic Monitor is enabled, otherwise the policy execution path will not be displayed in the **Traffic Monitor** options. If you only use the [protocols supported](#which-protocols-are-supported) by the Elastic solution, you can reduce the size of the OpsDB if you like, since the data comes from the Elasticsearch database.

## Can I use the component without an API Manager?

Yes. In this case, you must set the `API_MANAGER_ENABLED=false` parameter and disable [user authorization](/docs/operational_insights/additional_features/op_insights_custom_user_authz).

## Can I use the compoonent with Elasticsearch >=8.0.0

No. Elasticsearch `8.0.0` introduces a number of breaking changes and this component has not yet been updated to become compatible with those updates.

## Can we support the component in non-docker mode?

No. The component is solely designed to run based on Docker containers.

## Can we get rid of API Builder and instead leverage policies in API Gateway for API detail lookup and other requirements?

No. A large part of the logic of the component is in the API Builder application. Implementing this in policies might be possible, but managing and updating the individual customer installations would be very time-consuming and error-prone. So, you must reference the appropriate API Builder image.

## Can API Builder be made optional?

No. API Builder is a substantial part of the solution. It exposes the fully tested Traffic Monitor API, manages Elasticsearch, and provides Lookup APIs, all fully automated tested.

## Can we minimize the number of dependencies?

Elastic Search, Logstash, Kibana, and FileBeat agents are mandatory.

## Is it possible to use another caching solution than Memcached?

No. Logstash uses a filter to communicate with Memcached, and is integrated into the Logstash pipelines. Only the interaction of the pipelines and Memcached has been tested and is thus supported.

## Do I need to update the Elasticsearch version?

The component ships a defined Elasticsearch version with each release, which is used by all components of the Elastic stack. If nothing else is specified, then you can stay on the Elastic Stack version defined in your `.env` file.

## Can I skip several versions for an upgrade?

Yes, but be aware of which components have been updated between the current and the new version.

## Can I downgrade to a previous version?

No, you cannot downgrade from a newer version to an older one. You will see the following error message:

```none
cannot downgrade a node from version [7.16.1] to version [7.15.2]
```

Note that as soon as a newer Elasticsearch node is started, the data stored on the volume is updated, which makes it impossible to start an older version.

## Why the Helm release is named axway-elk?

The release name must currently be `axway-elk` because many resources, like services, ConfigMaps, or Secrets are created with it and referenced in the standard `values.yaml`. An example is the Elasticsearch-Service, `axway-elk-apim4elastic-elasticsearch`, which is for instance used for the standard elasticsearchHosts, `<https://axway-elk-apim4elastic-elasticsearch:9200>`.

## Can I use auto-scaling in Helm?

No. Elasticsearch automatically stores the data on multiple cluster nodes and would have to rebalance the cluster again and again when adding or removing nodes (pods). Logstash also does not offer the option of autoscaling. However, you can change the number of PODs for, lets say, Elasticsearch, Logstash at any time. The condition is that they do not change within a short time, otherwise no advantage is gained.

## Can I easily add or remove instances in Helm?

Yes, they can change the number of nodes via the scaling function or via their `values.yaml` using replicas. This works for APIBuilder4Elastic, Logstash, Elasticsearch, and Kibana.
For Elasticsearch, ensure the cluster is in `Green` state before removing a node to avoid data loss. If you change the Logstash instances, it is recommended to restart all Filebeat instances afterwards. The reason is the persistent connections. You might also need to adjust the filebeat configuration.

## How can I increase the disk space for Elasticsearch in Helm?

If Elasticsearch is running in Kubernetes as a StatefulSet, you can increase the disk space as follows.

1. Ensure that the StorageClass you are using supports `volumeExpansion. allowVolumeExpansion: true`.
2. Edit the existing PVC for already running PoDs and increase the capacity.
3. Update the VolumeClaimTemplate in your Helm values:
    ```bash
    volumeClaimTemplate:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: gp2
      resources:
        requests:
          storage: 100Gi
    ```
4. Delete the StatefulSet without deleting the pods:
    ```bash
    kubectl -n apim delete sts axway-elk-apim4elastic-elasticsearch --cascade=orphan
    ```
5. Perform a Helm upgrade to reinstall the modified StatefulSet.
6. Redeploy the PODs of the stateful set.
    ```bash
    kubectl -n apim rollout restart sts axway-elk-apim4elastic-elasticsearch
    ```
---
title: Troubleshooting Operational insights
linkTitle: Troubleshoot
weight: 200
date: 2022-06-09
description: Troubleshoot problems you might encounter when configuring and running Operational insights.
---

## Solution does not show data in real time

An important aspect of this capability is to store the API requests as fast as possible in Elasticsearch, so that they are available in real-time in the API Gateway Traffic monitor or in the Kibana dashboard.

If your setup cannot process the log information fast enough, then the solution may never be real-time and eventually even log information may be lost.

Watch [Axway API-Management Elasticsearch Integration - Performance Troubleshooting](https://www.youtube.com/watch?v=rLT84dkgGAE) video to learn why the document ingest rate might be too low, what information is available for analysis in Kibana, and how to correct this if necessary.

## Confirm that processes or containers are running

From within your GitHub project's folder where the `docker-compose.yml` file is located, run the following:

```bash
docker-compose inspect
```

The status of each container is shown depending on the services you enabled and disabled.

|                              Name                                 |            Command            |State        |Ports|
|---                                                                |---                            |---          |---  |
|apigateway-openlogging-elk_elk-traffic-monitor-api_1_3fbba4deea37  |docker-entrypoint.sh node     |Up (healthy)  |0.0.0.0:8889->8080/tcp|
|apigateway-openlogging-elk_filebeat_1_3ad3117a1312                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9000->9000/tcp|
|apigateway-openlogging-elk_logstash_1_c6227859a9a4                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:5044->5044/tcp, 9600/tcp|
|elasticsearch1                                                     |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp|

## Verify Filebeat is receiving data

You need to check the filebeat Log-File within the running docker container.

```bash
docker exec -it apigateway-openlogging-elk_filebeat_1_3ad3117a1312 bash
cd logs
tail -f filebeat
```

Verify the Filebeat harvester is started on the Open Traffic files:

```bash
INFO    log/harvester.go:251    Harvester started for file: /var/log/work/group-2_instance-1_traffic.log
```

The following error means that Logstash is not running or reachable, or just not yet fully started:

```bash
ERROR   pipeline/output.go:100  Failed to connect to backoff(async(tcp://logstash:5044)): lookup logstash on 127.0.0.11:53: no such host
```

{{< alert title="Note">}}
Filebeat does not shows a status when it is successfully processing your log files.
{{< /alert >}}

If you do not see any errors after the harvester process is started, you can assume your files are processed. If the harvester does not start, you might need to validate that the Open Traffic Event log files are visible by listing the following directory within the running container:

```bash
ls -l /var/log/work
-rw-rw-r--. 1 filebeat filebeat  2941509 Aug 13 12:38 group-2_instance-1_traffic.log
-rw-rw-r--. 1 filebeat filebeat 20972249 Jul  7 19:32 group-2_instance-1_traffic_2020-07-07-1.log
-rw-rw-r--. 1 filebeat filebeat 20972436 Jul  8 18:08 group-2_instance-1_traffic_2020-07-08-1.log
-rw-rw-r--. 1 filebeat filebeat 20972005 Jul 17 07:32 group-2_instance-1_traffic_2020-07-17-1.log
```

## Verify Logstash processing

Logstash writes to stdout, so you can view some log information just by running the following:

```bash
docker logs logstash -f
```

After Logstash is successfully started, you should see the following result:

```bash
docker logs logstash
Pipelines running {:count=>6, :running_pipelines=>[:".monitoring-logstash", :DomainAudit, :Events, :TraceMessages, :BeatsInput, :OpenTraffic], :non_running_pipelines=>[]}
Successfully started Logstash API endpoint {:port=>9600}
```

If you see the following or similar error message during processing of events, then API Builder lookup API cannot be reached. In this case, ensure the environment variable  `API_BUILDER_URL` is set correctly.

```bash
[2020-08-19T10:51:05,671][ERROR][logstash.filters.http    ][main][......] error during HTTP request {:url=>"/api/elk/v1/api/lookup/api", :body=>nil, :client_error=>"Target host is not specified"}
```

## Verify Admin node manager is running

If the Admin node manager (ANM) does not fetch the data from Elasticsearch by way of API Builder, it is recommended to check the ANM trace file. Possibly errors like the following are logged in the trace file:

```bash
ERROR   07/Dec/2020:01:43:19.769 [262e:27ebcd5f0601b4bda7f620b5]         java exception: 
java.lang.RuntimeException: java.net.URISyntaxException: Illegal character in path at index 0: [invalid field]/api/elk/v1/api/router/service/instance-1/ops/search?format=json&field=leg&value=0&count=1000&ago=24h&protocol=http
```

In this case, the environment variable `API_BUILDER_URL` is not set for the ANM process.

It can also be useful to use the Developer tools (Network monitor) in your browser (for example, Chrome or Firefox) to check the result of API requests in the Traffic Monitor. This way you can see the error straight away when API Builder fail to perform the request.

The following animations shows how to use the Developer tools:

* **add the gif here**

## Verify Elasticsearch processing

It takes a while until Elasticsearch is started. Run the following command to monitor when the process started:

```bash
docker logs elasticsearch1 --follow
```

When Elasticsearch is started you should see the following message:

```
"level": "INFO", "component": "o.e.c.r.a.AllocationService", "cluster.name": "elasticsearch", "node.name": "elasticsearch1", "message": "Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[.kibana_1][0]]]).", "cluster.uuid": "k22kMiq4R12I7BSTD87n5Q", "node.id": "6TVkdA-YR7epgV39dZNG2g"  }
```

Status YELLOW is expected when running Elasticsearch on a single node, as it cannot achieve the desired replicas. You might want to use Kibana Development tools or CURL to get additional information.

## Virtual memory is too low

The following error is show when virtual memory is too low:

```bash
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

Run the following to increase it temporarily:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Alternatively, you can also run:

```bash
sudo vi /etc/sysctl.conf
add vm.max_map_count=262144
sudo service sysctl restart
```

## No results from Elasticsearch

If you don't get any results from Elasticsearch for valid queries, an index template might not be applied correctly during index creation.

Be aware that Elasticsearch does not execute queries on the original document, rather on the indexed fields. How these were indexed is defined by an index mapping. For this purpose, this component delivers an index template for each index, which is used when the index is created.

You can find the index mapping in the API Builder container, `elasticsearch_config/<index-name>/index_template.json` or you can download it [here](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/apibuilder4elastic/elasticsearch_config).

To check wether the index mapping was applied correctly to an index, execute the following request. For example the Traffic-Summary index:

```bash
http://elasticsearch:9200/apigw-traffic-summary/_mapping
```

Then check whether properties mappings defined in the configuration mentioned above have been applied to the index.

## Monitor API Builder processing

To verify that API Builder docker container is running, run:

```bash
docker logs apibuilder4elastic --follow
```

If the container is running you should see the following message:

```bash
docker logs apibuilder4elastic --follow
```

## Verify requests from Admin node manager

placeholder

---
title: Troubleshooting for Operational Insights
linkTitle: Troubleshooting
weight: 300
date: 2022-06-09
description: Troubleshoot problems you might encounter when configuring and running Operational Insights.
---

## Verify data is flowing real-time

An important aspect of Operational Insights solution is to store the API requests as fast as possible in Elasticsearch, so that they are available in real-time in the API Gateway Traffic Monitor or in the Kibana dashboard. If your setup cannot process the log information fast enough, data will not be real-time and log information might be lost.

## Verify that processes or containers are running

From within your project's folder, where the `docker compose.yml` file is located, run the following:

```bash
docker compose inspect
```

The status of each container is shown depending on the services you enabled and disabled.

|                              Name                                 |            Command            |State        |Ports|
|---                                                                |---                            |---          |---  |
|apigateway-openlogging-elk_elk-traffic-monitor-api_1_3fbba4deea37  |docker-entrypoint.sh node     |Up (healthy)  |0.0.0.0:8889->8080/tcp|
|apigateway-openlogging-elk_filebeat_1_3ad3117a1312                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9000->9000/tcp|
|apigateway-openlogging-elk_logstash_1_c6227859a9a4                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:5044->5044/tcp, 9600/tcp|
|elasticsearch1                                                     |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp|

## Verify Filebeat is receiving data

Check the filebeat log file within the running docker container.

```bash
docker exec -it apigateway-openlogging-elk_filebeat_1_3ad3117a1312 bash
cd logs
tail -f filebeat
```

Verify the Filebeat harvester is started on the open traffic files:

```none
INFO    log/harvester.go:251    Harvester started for file: /var/log/work/group-2_instance-1_traffic.log
```

The following error means that Logstash is not running or reachable, or just not yet fully started:

```none
ERROR   pipeline/output.go:100  Failed to connect to backoff(async(tcp://logstash:5044)): lookup logstash on 127.0.0.11:53: no such host
```

{{< alert title="Note">}}Filebeat does not show a status when it is successfully processing log files.{{< /alert >}}

If you do not see any errors after the harvester process is started, you can assume your files are processed. If the harvester does not start, you might need to validate that the Open Traffic Event log files are visible by listing the following directory within the running container:

```bash
ls -l /var/log/work
-rw-rw-r--. 1 filebeat filebeat  2941509 Aug 13 12:38 group-2_instance-1_traffic.log
-rw-rw-r--. 1 filebeat filebeat 20972249 Jul  7 19:32 group-2_instance-1_traffic_2020-07-07-1.log
-rw-rw-r--. 1 filebeat filebeat 20972436 Jul  8 18:08 group-2_instance-1_traffic_2020-07-08-1.log
-rw-rw-r--. 1 filebeat filebeat 20972005 Jul 17 07:32 group-2_instance-1_traffic_2020-07-17-1.log
```

## Verify Logstash processing

Logstash writes to stdout, so you can view some log information by running the following:

```bash
docker logs logstash -f
```

If Logstash has successfully started, you should see the following result:

```none
docker logs logstash
Pipelines running {:count=>6, :running_pipelines=>[:".monitoring-logstash", :DomainAudit, :Events, :TraceMessages, :BeatsInput, :OpenTraffic], :non_running_pipelines=>[]}
Successfully started Logstash API endpoint {:port=>9600}
```

If you see the following or similar error messages during the processing of events, then API Builder lookup API cannot be reached. In this case, ensure the environment variable  `API_BUILDER_URL` is set correctly.

```none
[2020-08-19T10:51:05,671][ERROR][logstash.filters.http    ][main][......] error during HTTP request {:url=>"/api/elk/v1/api/lookup/api", :body=>nil, :client_error=>"Target host is not specified"}
```

## Verify Admin Node Manager is running

If the Admin node manager (ANM) does not fetch the data from Elasticsearch by way of API Builder, it is recommended to check the ANM trace file. Possibly errors like the following are logged in the trace file:

```none
ERROR   07/Dec/2020:01:43:19.769 [262e:27ebcd5f0601b4bda7f620b5]         java exception: 
java.lang.RuntimeException: java.net.URISyntaxException: Illegal character in path at index 0: [invalid field]/api/elk/v1/api/router/service/instance-1/ops/search?format=json&field=leg&value=0&count=1000&ago=24h&protocol=http
```

In this case, the environment variable `API_BUILDER_URL` is not set for the ANM process.

## Verify Elasticsearch processing

Run the following command to monitor when the process starts:

```bash
docker logs elasticsearch1 --follow
```

When Elasticsearch has started you should see the following message:

```none
"level": "INFO", "component": "o.e.c.r.a.AllocationService", "cluster.name": "elasticsearch", "node.name": "elasticsearch1", "message": "Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[.kibana_1][0]]]).", "cluster.uuid": "k22kMiq4R12I7BSTD87n5Q", "node.id": "6TVkdA-YR7epgV39dZNG2g"  }
```

Status `YELLOW` is expected when running Elasticsearch on a single node, as it cannot achieve the desired replicas. You might want to use Kibana Development tools or CURL to get additional information.

## Virtual memory

The following error is shown when virtual memory is too low:

```none
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

## No results observed from Elasticsearch

If you do not retrieve any results from Elasticsearch for valid queries, an index template might not be applied correctly during index creation.

Elasticsearch does not execute queries on the original document, rather on the indexed fields. How these were indexed is defined by an index mapping. For this purpose, Operational Insights delivers an index template for each index, which is used when the index is created.

You can find the index mapping in the API Builder container, `elasticsearch_config/<index-name>/index_template.json` or you can find it in the `apibuilder4elastic/elasticsearch_config` directory.

To check whether the index mapping was applied correctly to an index, execute the following request. For example the Traffic-Summary index:

```bash
http://elasticsearch:9200/apigw-traffic-summary/_mapping
```

Then, check whether properties mappings defined in the configuration mentioned above have been applied to the index.

## Verify API Builder processing

To verify that the API Builder docker container is running, run:

```bash
docker logs apibuilder4elastic --follow
```

If the container is running you should see the following message:

```none
docker logs apibuilder4elastic --follow
```

## Verify requests from Admin node manager

When using the API Gateway Traffic Monitor to monitor requests and having the Admin Node Manager reconfigured you should see how API-Builder is processing the requests:

```none
Request {"method":"GET","url":"/api/elk/v1/api/router/service/instance-1/ops/search?format=json&field=leg&value=0&count=1000&ago=10m&protocol=http","headers":{"host":"localhost:8889","max-forwards":"20","via":"1.0 api-env (Gateway)","accept":"application/json","accept-language":"en-US,en;q=0.5","cookie":"cookie_pressed_153=false; t3-admin-tour-firstshow=1; VIDUSR=1584691147-TE1M3vI9BFWgkA%3d%3d; layout_type=table; portal.logintypesso=false; portal.demo=off; portal.isgridSortIgnoreCase=on; 6e7e1bb1dd446d4cd36889414ccb4cb7=8g9p3kh27t1se22lu6avkmu0a1; joomla_user_state=logged_in; 220b750abfbc8d2f2f878161bab0ab65=62gr71dkre858nc0gjldri18gt","csrf-token":"8E96374767C47BFADC9C606FF969D7CF56FB3F9523E41B34F3B3B269F7302646","referer":"https://api-env:8090/","user-agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/74.0","x-requested-with":"XMLHttpRequest","connection":"close","x-correlationid":"Id-fd7c745ebfaed039b2155481 1"},"remoteAddress":"::ffff:172.25.0.1","remotePort":55916}

Response {"statusCode":200,"headers":{"server":"API Builder/4.25.0","request-id":"35fb859d-00b0-404b-97e6-b549db17f84c","x-xss-protection":"1; mode=block","x-frame-options":"DENY","surrogate-control":"no-store","cache-control":"no-store, no-cache, must-revalidate, proxy-revalidate","pragma":"no-cache","expires":"0","x-content-type-options":"nosniff","start-time":"1584692477587","content-type":"application/json; charset=utf-8","response-time":"408","content-md5":"e306ea2d930a3b80f0e91a29131d520b","content-length":"267","etag":"W/\"10b-2N+JsHuxDxMVKhJR1A8GuNGnKDQ\"","vary":"Accept-Encoding"}}
```

If you do not see any requests arriving in API builder, the ANM might not be able to reach the API builder listen socket, but traffic information will still be displayed because, in this case, the OpsDB will be used. In this scenario, check the ANM trace log.

## Check queries sent to ElasticSearch

To see queries that are sent to ElasticSearch by API Builder, you must run the Docker container with `LOG_LEVEL=debug`. You can activate debug in your `.env` file. This outputs the following in in the console of API Builder:

```none
Using elastic search query body: {"index":"logstash-openlog","body":{"query":{"bool":{"must":[{"range":{"timestampOriginal":{"gt":1587541496568}}},{"term":{"processInfo.serviceId":"instance-1"}}]}}},"size":"1000","sort":""}
```

This helps you to further analyze if ElasticSearch is returning the correct information, for instance, using the Kibana development console.

## ILM rollover alias error

If the solution is configured with different regions, you might see the following error in Kibana or in the Elasticsearch logs:

```none
index.lifecycle.rollover_alias [apigw-traffic-summary] does not point to index [apigw-traffic-summary-us-dc1-000002]
```

When an index is rolled, for example, because of the size of the configuration, time has been reached or because of an update, a new index is created based on the corresponding index template. In this template, the ILM rollover alias is configured, which does not match the index if it is regional. API Builder checks periodically (every 10 minutes) if the ILM rollover alias is correct and adjusts it if necessary. You can also start this modification manually:

```bash
docker exec apibuilder4elastic wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/index/rolloverAlias -O/dev/null
```

Additionally you may set the index rollover alias to the correct value using for example this request:

```json
PUT /apigw-trace-messages-eu-000001/_settings
{
  "index" : {
    "lifecycle.rollover_alias" : "apigw-trace-messages-eu"
  }
}
```

## Check caching

The solution uses Memcached in API Builder and Logstash to avoid unnecessary duplicate queries for APIs and users. To check if entries are cached correctly, you can connect to Memcached via telnet `localhost 11211`. The following are examples on how to use it.

### Get slabs

Memcache is storing the data into slabs.

```bash
stats cachedump 2 0
ITEM ignoredAPIs:###Health Check [19 b; 1610535266 s]
ITEM ignoredAPIs:/healthcheck### [19 b; 1610535266 s]
ITEM index_status:events###N/A [27 b; 1610538266 s]
ITEM index_status:trace###N/A [27 b; 1610538266 s]
ITEM index_status:openlog###N/A [27 b; 1610538266 s]
END
```

You might also try `stats cachedump 1 0` instead.

To get a key, for instance, the version check status, run:

```bash
get version_status:versionCheck
VALUE version_status:versionCheck 1 89
{I"
   message:ETI"'Filebeat and Logstash version okay;TI"versionStatus;TI"ok;T
END
```

For more information, see [Accessing Memcached from the command line](https://techleader.pro/a/90-Accessing-Memcached-from-the-command-line), and [PHPMemcachedAdmin](https://github.com/elijaa/phpmemcachedadmin) to get insights about the Memcache instance.

## Certificate error Admin Node Manager to API Builder

If you have errors connecting from the Admin Node Manager to the API Builder, ensure the Admin Node Manager has imported the correct CA. If you connect to API Builder using a hostname other than `apibuilder4elastic`, you might receive the following error message:

```none
The certificate was not issued by the domain issuer
```

To solve this issue, you can create a [remote host](/docs/apim_policydev/apigw_gw_instances/general_remote_hosts/) and disable the hostname validation (`Verify server’s certificate matches requested hostname`), or you can use your own matching keys and certificates).

Another reason for issues with connecting, might be a missing or expired license for the Admin Node Manager.

## Filebeat - Failure to publish events

If you see the following error message in the filebeat log, it means that Logstash could not accept any more events or that Elasticsearch cluster can not process any more events.

```none
failed to publish events: write tcp 172.23.0.2:55204->172.31.48.67:5044: write: connection reset by peer
```

The following list some options to check the issue:

* Check the CPU utilization of the Logstash nodes. Restart them if necessary.
* Update to randomise the logstash host order on every filebeat.
* Check that the Logstash instances have configured all available Elasticsearch nodes.
* Add more Logstash nodes on a test basis, if that doe not improve things, add an Elasticsearch node.

## Kibana - Missing long term statistics

If the Quarterly and Yearly API request processing dashboards do not display any data, check if the `apigw-traffic-summary-hourly-v<n>` transform job is running. The APIBuilder4Elastic application tries to create this job every hour. You can also start this with the following API request in the API Builder container:

```bash
docker exec apibuilder4elastic wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/transform/apigw-traffic-summary
```

If the job exists, check if the `apigw-hourly-traffic-summary-00000<n>` index exists. If not, stop the transform job and start it again. If necessary check the messages of the transform job. If the index exists but does not contain any documents, delete it and restart the transform job.

Finally, you can check if the `apigw-hourly-traffic-summary*` index pattern exists. If not, reimport the Kibana dashboard configuration, `Axway-api-overview.ndjson`.

If the **API status history** and **API Gateway status history** in the Quarterly or Yearly dashboards are not shown, perform the following.

* Stop and delete the transform job: `apigw-traffic-summary-hourly-v<n>`
* Delete the hourly index: `apigw-hourly-traffic-summary-00000<n>`

Then, either wait for, maximum, 60 minutes until API Builder has recreated the transform, or call the REST API as shown above.

The main cause of this problem is the way the `http.status` field is indexed. If no documents have been received after the upgrade and initial creation of the transformation job, the transformation does not correctly index the `http.status.keyword` field.

## Too many API Manager lookups

API and Application details are looked up as part of the Logstash pipeline from the API Manager using a APIBuilder4Elastic REST API. The obtained details are cached by Logstash using Memcache.
If lookups are regularly performed, this might be caused by APIs which have a path parameter (for example, `/api/v1/cusomer/323213`). Because the path parameter makes the API path dynamic, it prevents Logstash from caching it. In this case, you need to help the implementation by setting up the parameter, `CACHE_API_PATHS`.

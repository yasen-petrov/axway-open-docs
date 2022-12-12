---
title: Configure local lookup
linkTitle: Configure local lookup
weight: 40
date: 2022-09-28
description: Use local configuration files to enhance logs.
---

You can use Operational Insights local configuration files for the API lookup in addition to the API Manager.

The following are some of the advantages of enhancing your logs:

* Enrich APIs that have been exposed natively through the API Gateway policies not having a context.
* You can configure all information that would normally be retrieved from API Manager (Organization, API Name, API method name, custom properties) by way of this lookup file. For example, you can display `/healthcheck` as a **Healthcheck** API in Kibana dashboards.
* You can ignore OpenTraffic events so that they are not indexed or stored in Elasticsearch. For example, the `/favicon.ico` path can generally be ignored, as this event does not usually add value.

Note that the local configuration file is used before the API Manager lookup. If there is a match, no lookup to the API Manager is performed.

## Enable local lookup

To enable the local lookup, perform the following steps:

1. Add your config file. It is best to copy the delivered template, `config/api-lookup-sample.json`, to your `config/api-lookup.json` file.

    ```bash
    cp config/api-lookup-sample.json config/api-lookup.json
    ```

2. In your `.env` file, configure the following environment variable to enable the configuration file to be used by API Builder:

    ```bash
    # For docker compose use case set the env variable and restart:
        API_BUILDER_LOCAL_API_LOOKUP_FILE=./config/api-lookup.json
        docker stop apibuilder4elastic
        docker compose up -d
    ```

    ```bash
    # For helm use case expose variable localAPILookup in **myvalues.yaml** file and run helm upgrade:
        localAPILookup: "./config/api-lookup.json"
        helm upgrade -n apim-elk -f myvalues.yaml axway-elk axway/aaoi-helm-prod
    ```

    If an event is to be indexed, API builder will try to read this file and will acknowledge this with the following error if the file cannot be found:

    ```bash
    Error reading API-Lookup file: './config/api-lookup.json'
    ```

### Ignore events

At this point, we intentionally refer to events and not APIs, because different events (TransactionSummary, CircuitPath, TransactionElement) are created in the OpenTraffic log for each API call. Each is processed separately by Logstash and stored using an `upsert` in Elasticsearch with the same correlation ID. Most of these events contain the path of the called API. This is especially important when ignoring events so that they are not stored in Elasticsearch entirely. Because all events are processed individually, it is therefore decided on an individual basis which events to ignore.

To ignore, for example, the Healthcheck API entirely, the following must be configured in the lookup file:

```bash
{
    "/healthcheck": {
        "ignore": true,
        "name": "Healthcheck API",
        "organizationName": "Native API"
    },
    "Policy: Health Check": {
        "ignore": true
    }
}
```

This will ignore events based on the path (TransactionSummary and TransactionElement) and the policy name (CircuitPath). You can see the result in the API Builder log, on level INFO, with the following lines:

```log
Return API with apiPath: '/healthcheck', policyName: '' as to be ignored: true
```

or

```log
Return API with apiPath: '', policyName: 'Health Check' as to be ignored: true
```

The information is cached in Logstash via memcached for one hour, so you will not see the log in API Builder for each request. But, you can force a reload of updated configuration by running `docker restart memcached`.

#### Ignore events by region and group

It is additionally possible to overlay the local lookup file per group and region. This allows you, for example, to return different information per region for the same native API or to ignore an API only in a specific region. To do this, create files with the same base name, then a qualifier for the group or for the group plus region. For example:

```
api-lookup.group-2.json
api-lookup.group-2.us.json
```

Again, it is not possible to specify only the region, but only in combination with the appropriate group.

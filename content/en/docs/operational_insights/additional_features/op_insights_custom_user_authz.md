---
title: Customize user authorization
linkTitle: Customize user authorization
weight: 30
date: 2022-09-28
description: Customize user authorization in your environment to view traffic in API Gateway Manager.
---

The organization of the API Manager user determines authorization to the Traffic Monitor by default. This means that the user only sees traffic from the organizations to which they belong in API Manager. This is done by attaching an additional filter clause to the Elasticsearch query. This extra filter restricts the returned data, so only the data that belongs to the API users organization is retrieved.

The following sections describe how to customize user authorization in your environment.

### Customize user authorization in Docker Compose

You can use an external HTTP service for authorization instead of the API Manager organizations to restrict the Elasticsearch result based on other criteria. To customize user authorization in Docker Compose, you must create a configuration file as follows:

```bash
# Copy the provided example from the docker release package
cp ./config/authorization-config-sample.js ./config/myAuthzConfig.js
# Customize your configuration file as needed
vi ./config/myAuthzConfig.js
# Setup your .env file to use your authorization config file
vi .env
AUTHZ_CONFIG=./config/myAuthzConfig.js
# Recreate the API Builder container
docker stop apibuilder4elastic
docker compose up -d
```

In this configuration, which also contains corresponding Javascript code, required parameters and code are stored, for example, to parse the response and to adjust the Elasticsearch query. For more information, see the `config/authorization-config-sample.js` example file in the downloaded package. After this configuration is stored, the API Manager Organization based authorization is replaced.

When customizing user authorization, observe the following:

* Besides the API Manager Organization authorization only `externalHTTP` is currently supported.
* Only one authorization method can be enabled.
* It is also possible to disable user authorization completely. To do this, set the `enableUserAuthorization` parameter to `false` in your `env` file.

### Customize user authorization using Helm

Follow these steps to customize user authorization using Helm:

1. Create a ConfigMap that creates your custom configuration file. For more information, see the `config/authorization-config-sample.js` sample file.

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: apibuilder4elastic-authz-config
    data:
      myAuthzConfig.js: |
        const path = require('path');
        const fs = require('fs');
    
        /*
            By default, the solution uses user's API Manager organization to determine which 
            API-Requests they are allowed to see in the API Gateway Traffic-Monitor. 
            This behavior can be customized. 
        */
    
        var authorizationConfig = {
            // For how long should the information cached by the API-Builder process
            cacheTTL: parseInt(process.env.EXT_AUTHZ_CACHE_TTL) ? process.env.EXT_AUTHZ_CACHE_TTL : 300,
            // If you would like to disable user authorization completely, set this flag to false
            enableUserAuthorization: true,
            // Authorize users based on their API-Manager organization (this is the default)
            apimanagerOrganization: {
                enabled: true
            },
        ....
        ..
        .
    ```

2. Optionally, you can change the generated YAML file and use it as a Helm template.

    You can use `.Files.get.` to include your custom configuration file. For more information, see `templates/elasticApimLogstash/logstash-pipelines.yaml`.

3. Install or upgrade your setup chart:

    ```
    helm upgrade -n apim-elk axway-elk-setup .
    Release "axway-elk-setup" has been upgraded. Happy Helming!
    NAME: axway-elk-setup
    LAST DEPLOYED: Tue May  4 15:06:30 2021
    NAMESPACE: apim-elk
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    ```

4. Mount the ConfigMap into the API Builder container and reference it in the configuration using the `values.yaml`:

    ```yaml
    apibuilder4elastic:
      extraVolumes:
      - name: my-authz-config
        mountPath: /app/config
        subPath: myAuthzConfig.js
        
      extraVolumeMounts:
        - name: my-authz-config
          configMap:
            name: apibuilder4elastic-authz-config
    ```

5. Configure APIBuilder4Elastic to use your custom configuration:

    ```yaml
    apibuilder4elastic:
      authzConfig: "./config/myAuthzConfig.js"
    ```

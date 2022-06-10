---
title: Configure API Gateway Manager for Elasticsearch
linkTitle: Configure API Gateway Manager
weight: 40
date: 2022-06-09
description: Configure API Gateway Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances.
---

After (some previous section), you must configure your API Gateway Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. By default, the API Gateway Manager API listens to port 8090 for administrative traffic. This API is responsible to serve the Traffic Monitor and it needs to be configured to use the API Builder REST API instead.

To configure API Gateway Manager to render log data from Elasticsearch, follow these steps:

1. Open the API Gateway Manager configuration in Policy-Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
3. To be continued...

## Next steps

On the following section, you are going to ...
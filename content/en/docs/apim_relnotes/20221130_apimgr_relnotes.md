---
title: API Gateway and API Manager 7.7 November 2022 Release Notes
linkTitle: API Gateway and API Manager November 2022
weight: 95
date: 2022-09-14
description: null
---
API Gateway and API Manager updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

We are excited to bring you an updated deployment methodology for containerised deployment, which uses a Helm chart and premade images in order to offer the most simple way to deploy the API Management solution, including the externalisation of the configuration of the API Gateway. Customers can now use our premade hardened images, and with the externalisation of the configuration, you don't need to rebake images when a change to the configuration is made. This makes the deployment and operation of the solution easier to manage.

We have also made an update for customers using HSMs with OpenSSL, moving from `engine` to `provider` API for PKCS11 communication with HSMs.

## Installation

* To **update** your API Gateway, see [Update from API Gateway One Version](/docs/apim_installation/apigw_upgrade/upgrade_steps_oneversion/).
* To **upgrade** from an older version, see [Upgrade from API Gateway 7.5.x or 7.6.x](/docs/apim_installation/apigw_upgrade/upgrade_steps_extcass/).
* For more details on supported platforms for software installation, see [System requirements](/docs/apim_installation/apigtw_install/system_requirements/).
* For a summary of the system requirements for a Docker deployment, see [Set up Docker environment](/docs/apim_installation/apigw_containers/docker_scripts_prereqs/).

### Update a container deployment

Any custom `.fed` files deployed to a container must be upgraded using [upgradeconfig](/docs/apim_installation/apigw_upgrade/upgrade_analytics#upgradeconfig-options) or [projupgrade](/docs/apim_reference/devopstools_ref#projupgrade-command-options). They must be upgraded the same way, regardless of whether they are API Manager enabled or not. The `.fed` files contain the updates for the API Manager configuration and can be used to build containers.

## New features and enhancements

The following new features and enhancements are available in this update.

### New API Gateway Docker image

A newly introduced API Gateway Docker image has been introduced, and is downloadable from the [Amplify repository](https://repository.axway.com). FIPS mode is not yet supported.

To get started, see [Deploy API Gateway with Axway Docker images](/docs/apim_howto_guides/configuring_apigw_container).

### Helm use of externalization

Axway Helm charts can now be used to externalize API Gateway configuration for containers. For more information, see [Deploy API Gateway using Helm](/docs/apim_howto_guides/container_helm_deployment) and [Deploy API Gateway with Axway Docker images](/docs/apim_howto_guides/configuring_apigw_container).

### Amplify Analytics Operational Insights

Amplify Analytics Operational Insights (AAOI), previously known as the Axway API Gateway (APIGW) ELK implementation project, has been migrated to the [Amplify repository](https://repository.axway.com). This component, which uses an ELK stack to better monitor traffic, can be set up using either Helm or Docker Compose.

If you have previously used the APIGW ELK implementation project, ensure you read over all documentation as stricter security options are now enabled by default.

For all information regarding this component, see [Amplify Analytics Operational Insights](/docs/operational_insights).

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update, which may impact your current installation.

### API Manager settings, Remote hosts input and output encodings can now be disabled

API Manager settings, Remote hosts can now be configured with no **Input encodings** or no **Output encodings** by toggling off the **Use default** switch (note that **deflate** and **gzip** switches are also off). This means no encodings to be used in line with Policy Studio interface and entity store `RemoteHost` configuration (an empty array).

For more information, see [API Manager settings, Remote hosts](/docs/apim_reference/api_mgmt_config_web#remote-hosts).

### Changes in OAuth Login Form path

The sample **Login** form provided with OAuth configuration contains a hidden field called `continue`. The value for this field has changed from a URL to a URI path. Any policy that processes this form and validates this field should be updated accordingly.

For more information, see [Login sample policy](/docs/apim_policydev/apigw_oauth/gw_oauth_authz_server#login-sample-policy).

### Traffic Monitor filter behavior change with threat protection

On Traffic Monitor, when the **Transaction Status** filter is set to **Block** any failed transactions through the API Gateway are displayed. There has been a change to this behavior to now also match any transactions that were blocked by threat protection on request.

Previously, transactions blocked by threat protection on request would not be displayed on Traffic Monitor when redirected from the **Blocked** widget due to them not having an associated transaction status. With this update, the **Transaction Status** filter will additionally display these transactions.

For more information on filters and real-time monitoring in Traffic Monitor, see [Monitoring and metrics](/docs/apim_administration/apigtw_admin/monitor_service#view-traffic-monitoring).

### API Manager no longer allows exporting front-end APIs as plain text by default

The **Export API** feature, in the API Manager, now requires a mandatory password to encrypt the file containing the exported APIs collection by default.

To allow exporting front-end APIs as plain text, you must set the `com.axway.apimanager.api.export.cleartext.allowed` system property to `true`. This is not recommended in production environments, because any confidential keys will be clearly visible if the file containing the exported APIs collection is not password protected.

For more information, see [Manage front-end REST API lifecycle, Encryption of exported API collections](/docs/apim_administration/apimgr_admin/api_mgmt_virtualize_web#encryption-of-exported-api-collections) and [System property changes, 7.7 November 2022](/docs/apim_reference/system_props#77-november-2022).

### New checkbox in the Advanced (SSL) tab of the Connect To URL filter

There is a new checkbox in the Advanced (SSL) tab of the **Connect To URL** filter. When selected, the checkbox enables unsafe legacy renegotiation, which allows initial SSL connections and renegotiation with servers that do not advertise support for Renegotiation Indication Extension (RFC 5746). For more information, see [Configure SSL settings](/docs/apim_policydev/apigw_polref/routing_common#configure-ssl-settings).

### Runtime parameter validation system properties have been replaced with API Manager settings

System properties were previously introduced to the product to allow three types of validation to be disabled at runtime during API method invocation. The following properties have been replaced with API Manager system settings:

* `com.axway.api.runtime.broker.parameters.skipRequiredValidation` has been replaced with the **Skip required validation** setting.
* `com.axway.api.runtime.broker.parameters.skipEnumValidation` has been replaced with the **Skip enum validation** setting.
* `com.vordel.coreapireg.runtime.broker.parameters.allowEmptyDefault` has been replaced with the **Allow empty value** setting.

Customers using these system properties must configure the corresponding API Manager settings during update. For more information, see the following in API Gateway documentation:

* [System property changes in 7.7 November 2022](/docs/apim_reference/system_props#77-november-2022).
* [Runtime parameter validation section in API Manager settings](/docs/apim_reference/api_mgmt_config_web#runtime-parameter-validation).

### EdDSA certificates are supported

TLS and PKI certificates using EdDSA signatures (`ed25519` and `ed448`) for authentication are now supported within API Gateway. Certificates can be imported into the `config` file and used in TLS listeners, and connections can be established with external endpoints that use EdDSA certificates. For more information, see [Configure TLS authentication with API Manager](/docs/apim_installation/apiportal_install/mutual-tls-authentication-with-api-manager/)

## Deprecated features

No features have been deprecated in this update.

## End of support notices

There are no end of support notices in this update.

## Removed features

<!--To stay current and align our offerings with customer demand and best practices, Axway might discontinue support for some capabilities. As part of this update, the following features have been removed:-->

No features have been removed in this update.

## Fixed issues

This version of API Gateway and API Manager includes:

* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [Release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220830/ipp/100/product/324/product/464/version/3034/version/3035/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

| Internal ID | Case ID                                | Cve Identifier | Description                                                                                                                                                                                                                                                                                     |
| ----------- | -------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|RDAPI-27282|01364584||**Issue**: API Manager front-end API dat file exported can be plain text via API. **Resolution**: Exporting front-end APIs now requires a mandatory password to encrypt exported APIs collection by default. A new 'com.axway.apimanager.api.export.cleartext.allowed' system property was added for backward compatibility purposes and is not recommended in production. If set to 'true', the API Manager allows exporting front-end APIs as clear text,  and the password parameter is considered optional. Defaults to 'false'.|
|RDAPI-27943|01387610||**Issue**: The login form and related policy provided in the OAuth sample config contained an open redirect bug **Resolution**: The open redirect has been removed in the sample OAuth policy|
|RDAPI-28320|01401127||**Issue**: When self-service is enabled, the list front-end APIs via GET /proxies API for an Organization Administrator user in one or more organizations returns all front-end APIs regardless of user membership and role in other organizations. **Resolution**: The list front-end APIs via GET /proxies API is now returning only the APIs for the organizations the user is an Organization Administrator of, regardless whether self-service is enabled or not.|
|RDAPI-28469|01405990|CVE-2022-40152 CVE-2022-40153 CVE-2022-40154 CVE-2022-40155 CVE-2022-40156|**Issue**: The API Gateway contains a Woodstox library version that is vulnerable to CVEs: CVE-2022-40152, CVE-2022-40153, CVE-2022-40154, CVE-2022-40155,CVE-2022-40156 **Resolution**: The Woodstox library is now upgraded to a version that is not vulnerable to these CVEs.|
|RDAPI-28528|01409861  01409938  01409428  01409037  01409313|CVE-2022-42889|**Issue**: Apache Commons-Text CVE-2022-42889. **Resolution**: Apache Commons-Text updated to 1.10.0 version.|
|RDAPI-28590|01413038  01413489  01412549  01412449  01413970|CVE-2022-3786  CVE-2022-3602|**Issue**: OpenSSL 3.0.0-3.0.5 have high CVE's published. **Resolution**: Updated to OpenSSL 3.0.7|
|RDAPI-28593|01412518|CVE-2022-40674|**Issue**: Libexpat prior to version 2.4.9 had a security vulnerabiity CVE-2022-40674 **Resolution**: Libexpat has been updated to version 2.4.9|
|RDAPI-28681|01414999|CVE-2022-21626|**Issue**: JRE 8u345 contained several CVEs **Resolution**: Gateway has updated the JRE to version 8u352|

### Other fixed issues

| Internal ID | Case ID                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|RDAPI-19442|01131896  01145304  01145024|**Issue**: When running two browser sessions either with different users or the same user in API Manager, multiple identical error dialogs are displayed on the screen when the user is trying to access an element (application/API) that is already removed or the access to it is denied. Identical dialogs are also displayed when refreshing a list view screen where an element is already removed. **Resolution**: Now, only one error dialog is displayed when the user tries to access an element that is already removed or the access to it is denied. When refreshing a list view screen where an element is already removed, no error dialog is shown.|
|RDAPI-19601|01140600|**Issue**: Changing an Application's Organization doesn't remove users that aren't associated with the new Organization. **Resolution**: Application sharing permissions are removed for users that do not belong to the Application's new Organization.|
|RDAPI-22198|01203849  01388511  01202919  01119847  01373200  01307955|**Issue**: The projugrade script adds the GeneratedPolicies container to projects as default. This can cause conflicts when merging projects. **Resolution**: A new option has been added to projupgrade, --noGeneratedPolicies, which will not add the default GeneratedPolicies container to projects if specified.|
|RDAPI-24148|01309360  01393825  01258866  01269927  01268921  01256706  01228890  01281231  01327649  01257081  01378887  01253734  01276115  01278358  01342065  01290154|**Issue**: API Manager runtime parameter validation reports errors that are not valid. This issue primarily affects parameters that are overridden with an outbound profile. However, some parameter definitions will also result in errors without an outbound profile being configured. Errors in question include required parameters being reported as missing when they are present, enum parameters reported as being invalid when they are valid, and parameters being reported as empty when they contain data. Selectors configured for parameters in outbound profiles sometimes do not resolve correctly at runtime as well. API methods registered in Policy Studio can also encounter an issue where required parameters are not validated, and requests are processed even if they are missing. Parameters of type 'multipart/form-data' were also case sensitive by default, which is incorrect. </br>**Resolution**: Runtime parameter validation now operates correctly in all scenarios. The following system properties were also removed from the product and replaced with API Manager system settings: com.axway.api.runtime.broker.parameters.skipRequiredValidation, com.axway.api.runtime.broker.parameters.skipEnumValidation and com.vordel.coreapireg.runtime.broker.parameters.allowEmptyDefault.|
|RDAPI-25390|01342873  01288025|**Issue**: API Manager might use a significant percentage of Cassandra nodes CPU when creating, upgrading, and deprectating virtualized APIs in some systems. **Resolution**: API Manager REST APIs have been updated to reduce communication with Cassandra, hence reducing load on Cassandra.API Manager caching mechanism has been improved and expanded to cache additional Cassandra tables when the 'com.axway.apimanager.api.data.cache' system property is set to true.This improves the response times of the following API Manager REST APIs </br>- API Proxy Registration APIs </br>- API Repository APIs </br>- The following Applications APIs </br>-- APIs dealing with External Clients </br>-- APIs dealing with OAuth Client IDs  </br>-- APIs dealing with OAuth Resources </br>-- APIs dealing with API Accesses </br>-- Organizations APIs dealing with API Accesses </br>-- Migrate APIs </br>-- API Manager Services dealing with Remote Hosts. </br>Note that previously, API Manager attempted to cache all user, organization, application, and application permission items, which could lead to memory exhaustion in API Manager and out of memory errors. The new API Manager caching mechanism limits the cache to 100000 items in-memory per KPS table accessed by the above API Manager REST APIs, and when a KPS table size exceeds the 100000 allowed items, then API Manager falls back to access the items directly from the corresponding KPS table bypassing the cache to avoid exhaustion of resources.|
|RDAPI-25522|01357657  01397129  01297263|**Issue**: In API Manager, if an API that has been imported is cloned and the original back-end API is subsequently deleted, the 'Download original API definition' option is still available after a restart of the API Gateway process, resulting in a '500 Internal Server Error' response from the server. </br>**Resolution**: When an API that has been imported is cloned, and the original back-end API is subsequently deleted, the 'Download original API definition' option is no longer enabled, thus preventing the server from returning the '500 Internal Server Error' response. When the original back-end API is deleted, the hasOriginalDefinition flag on all clones is set to FALSE. This also holds true for clones that existed before applying this fix; if clones exist where the original back-end API has been deleted, a check is performed to see if the original definition is available and if it isn't, the hasOriginalDefinition flag is set to FALSE, disabling the 'Download original API definition' option and preventing a '500 Internal Server Error' from occurring. A similar check is performed for clones being imported as part of an API Collection.|
|RDAPI-25685|01235802|**Issue**: Duplication of an empty clientId was possible when modifying external clients. **Resolution**: Empty clientId duplication is no longer allowed.|
|RDAPI-26122|01318616|**Issue**: Calling an API with OPTIONS verb to an existing path generates a response with a status of 404 instead of 204. **Resolution**: Calling an API with OPTIONS verb responds 204 with an 'Allow' header with allowed verbs when the path exists. It responds 404 when the path does not exist.|
|RDAPI-27001|01350049|**Issue**: Issues loading APIs to the catalog after updating from 7.7 SP2 to 7.7 November 2021 release. **Resolution**: The APIs are now successfully loaded to the catalog after updating from a release prior to 7.7 January 2020.|
|RDAPI-27422|01369438|**Issue**: PKCS11 layer does not set the CKF_OS_LOCKING_OK flag when initializing while multi-threaded calls are made from the product. **Resolution**: PKCS11 layer sets the CKF_OS_LOCKING_OK on initialize. If the HSM device does not support the flag, all API calls will be serialized and not executed simultaneously.|
|RDAPI-27445|01369123|**Issue**: Optional parameters being removed from application/x-www-form-urlencoded Content-Type header in API Manager runtime broker. **Resolution**: API Manager can now be configured to preserve attributes of the 'application/x-www-form-urlencoded' Content-Type header of user request messages to API Manager by setting the new `com.axway.api.runtime.broker.contentType.formUrlEncoded.preserve` Java system property to `true`. Defaults to `false`.  |
|RDAPI-27594|01375542  01400128|**Issue**: Some confusion existed around the signing algorithm employed by API Manager's AWS security devices **Resolution**: The documentation has been updated to clarify the signing algorithm employed by API Manager's AWS security devices. For more information, see [Configure inbound request settings](/docs/apim_administration/apimgr_admin/api_mgmt_virtualize_web#configure-inbound-request-settings)|
|RDAPI-27669|01375686|**Issue**: Transactions blocked by Threat Protection on request are not visible on the API Gateway Manager dashboard. **Resolution**: Transactions blocked by Threat Protection on request are now visible on the API Gateway Manager dashboard.|
|RDAPI-27716|01379227|**Issue**: When a project with PGP filters is loaded by Policy Studio, there are problems with instantiating the PGP 'Encrypt and Sign' and 'Decrypt and Verify' filters. **Resolution**: The issue with instantiating the PGP filters is fixed, and these filters are now loaded successfully.|
|RDAPI-27721|01376462|**Issue**: It was not possible to specify how new MimeTypes should be processed **Resolution**: It is now possible to pick a Body handler for new MimeTypes|
|RDAPI-27814|01380592|**Issue**: On transaction completion, an error "loopback disconnected" is raised in an internal service call by the RealTime Monitoring service. **Resolution**: The loopback connection used by the internal service call is now cleanly disconnected when both ends have finished their tasks.|
|RDAPI-27915|01386005|**Issue**: In API Manager's Organizations screen, when an attempt is made to display users for a specific Organization by clicking the User filter, it does not work as expected. **Resolution**: The User filter has now been fixed so that the users belonging to the Organization in question are displayed when the User filter is clicked.|
|RDAPI-27923|01371016  01386495|**Issue**: Throttling filter defaults to Floating Time Window option. This is a legacy option that should not be used. **Resolution**: Throttling filter now defaults to Smooth Rate Limiting.|
|RDAPI-28022|01389402|**Issue**: The Amplify Analytics Operational Insights component blocks a non-admin user from seeing JMS traffic. **Resolution**: Fixed apibuilder4elastic API to allow users to see JMS traffic when disabling the user authorization by setting the parameter enableUserAuthorization to false.|
|RDAPI-28148|01395749|**Issue**: RSA encryption was causing a crash in API Gateway due to null input value. **Resolution**: API Gateway crashing is fixed by proper handling of the case when input is null during RSA encryption.|
|RDAPI-28162|01410075  01396459  01391228  01397264|**Issue**: Connections to servers that do not advertise support for Renegotiation Indication Extension (RFC 5746) would be automatically rejected **Resolution**: A new checkbox in the Advanced (SSL) tab of the Connect To URL filter allows these connections to be selectively enabled.|
|RDAPI-28179|01391307|**Issue**: SSO User's Name in API Manager is not being updated with updated Identity Provider information **Resolution**: API Manager User's Name is now updated with the Identity Provider information at user login.|
|RDAPI-28181|01411396  01396548|**Issue**: Frontend API resource path case matching problem after upgrade. **Resolution**: Frontend API resource path is now case sensitive matching.|
|RDAPI-28196|01393043|**Issue**: Single Sign-On API Manager signing certificates are not being refreshed when an Identity Provider certificate rollover occurs. **Resolution**: Updated Identity Provider certificates are now periodically updated in the API Manager certificate store for Single Sign-On Assertion signature validation.|
|RDAPI-28230|01396414|**Issue**: Signature verification was causing a crash in API Gateway due to a null value of the supplied signature. **Resolution**: API Gateway crashing is fixed by proper handling of the case when signature is null during signature verification.|
|RDAPI-28291|01398958|**Issue**: API Manager users that have had their password reset are still able to invoke API Manager REST APIs without the need to change their password, meaning the temporary password emailed to them as part of the password reset process is deemed valid. **Resolution**: API Manager users that have had their password reset now must change their password before they can invoke API Manager REST APIs. This is now the default behaviour. A new JVM.XML system property, com.axway.apimanager.bypass.temporary.password.check, has been introduced (for backward compatibility) to circumvent checks done on temporary passwords. The recommendation is that the temporary password checking should be performed, otherwise it poses a security risk.|
|RDAPI-28304|01394271|**Issue**: The root cause of this issue is related to how the docker container handles the entity store meta-inf from the externalized entity store configuration. When the externalized entity store is loaded into the Gateway, the meta-inf was lost. This information is used by Policy Studio to validate the archive. **Resolution**: When either a fed or yaml based entity store is loaded from externalized configuration, the Gateway container will now replace its entity store meta-inf with the incoming configuration.|
|RDAPI-28318|01382741  01396675|**Issue**: The initial quota snapshot table creation in Cassandra uses the wrong consistency level (SERIAL), even if another consistency level is configured in API Gateway. **Resolution**: Ensure Cassandra always uses the configured consistency level in API Gateway.|
|RDAPI-28328|01401297|**Issue**: The Copy / Modify Attribute filter would throw an exception when encountering an attribute without a value  **Resolution**: The Copy / Modify Attribute filter no longer throws an exception and places a null value on the message with the defined attribute name.|
|RDAPI-28381|01401877|**Issue**: When renaming containers in Policy Studio and using YAML configuration, the entities are not correctly moved/updated. **Resolution**: Now, when renaming containers from Policy Studio with YAML configuration, the entities are now correctly moved/updated.|
|RDAPI-28383|01404311|**Issue**: Package properties are reset when a project is updated using Policy Studio. Package properties added, deleted, or modified are lost. **Resolution**: Package properties added, deleted, or modified are kept when a project is updated using Policy Studio.|
|RDAPI-28389|01404908|**Issue**: JWT tokens encrypted using OAEP with Sha256 are not compatible with 3rd party software. **Resolution**: OAEP encryption of Vordel provider has been corrected to use by default the same MGF1 digest algorithm as the signature digest algorithm.|
|RDAPI-28405|01405274|**Issue**: Changes to user validation were breaking updates to SSO user's profile when authentication and authorization were separated for SSO. **Resolution**: API Manager user profile is now validated correctly for updates to SSO user profiles with separated authentication and authorization.|
|RDAPI-28490|01404330|**Issue**: A threat protection policy added to an SSO enabled API Manager would cause SAML assertions to be removed before validation. **Resolution**: Threat protection policies no longer consume SAML assertions and can be safely added to API Manager.|
|RDAPI-28496|01406965|**Issue**: The 7.7.20220830 update was incorrectly removing required libraries which was preventing API Gateway Analytics from starting. **Resolution**: The required libraries have been reinstated and the API Gateway Analytics process now starts successfully. The [KB article #182363](https://support.axway.com/kb/182363/language/en) describes a workaround to this issue.|
|RDAPI-28507|01407894|**Issue**: API Manager cannot disable input/output encodings for Remote Host. **Resolution**: API Manager can now disable input/output encodings for Remote Host.|
|RDAPI-28520|01413327  01388666|**Issue**: LDAP connections timeout issue. **Resolution**: LDAP connections are closed if a communication exception is raised when trying to reuse a cached connection or if the connection is reused after the configured cache timeout (existing behaviour). Now, the connections are also released on a configuration update, e.g. deploying a new configuration to the API Gateway server.Also, as defined in the LDAP v3 ([RFC 2251](http://www.ietf.org/rfc/rfc2251.txt)), the LDAP authentication processor is registered to handle the _Notice of Disconnection_ to close the affected connection if notified (note that the LDAP server has to support and be configured to generate such notice of disconnection)|
|RDAPI-28524|01406968  01409180  01408147|**Issue**: The duration of leg 0 in Transaction Event Log record is always 0, while corresponding Traffic Monitoring record contains the correct value. **Resolution**: Transaction Event Log records are populated after Traffic Monitoring records, making values correctly stored.|
|RDAPI-28617|01413359  01413630|**Issue**: Multi Organization Administrators are unable to view all Organizations when granting access to APIs and are unable to revoke access to Organizations they are members of. **Resolution**: Multi Organization Administrators now see all Organizations when granting access to APIs, and when revoking access, they will see all Organizations except the Organization that owns the API.|

## Known issues

The following are known issues for this update.

### API Analytics PDF reports do not display chart contents

In API Analytics, the PDF reports no longer correctly display the contents of the charts. This issue has arisen due to a security upgrade of the `Highcharts.js` charting library. We are working on the fix of this functionality, to be released in a future update of API Gateway.

Related Issue: RDAPI-27301

### Scripting filter whiteboard attributes not preloaded for Jython scripts

The Scripting filter now uses a Jython 2.7 scripting environment (previously, Jython 2.5) to execute Jython scripts. As a result of this version change, the whiteboard attributes, such as `http.request.uri` and `http.request.verb`, are no longer preloaded for use by Jython scripts. However, you can run a Jython script to load these attributes before they are accessed as follows:

```
from com.vordel.trace import Trace

def invoke(msg):
    msg.forceGenerateAttributes()
    Trace.info("This trace statement was generated in script filter!  [" + str(msg.get("http.request.verb")) + "] [" + str(msg.get("http.request.uri")) + "]")
    return True
```

Related Issue: RDAPI-21363

### Quotes in filter name result in circuit path not being displayed in traffic monitor

In a YAML configuration, when a request is sent through an API Gateway that executes either a policy container or a filter, which names contain a double quotation mark character ("), the resultant transaction cannot be viewed correctly in the API Gateway Manager Traffic Monitor. This issue occurs because the circuit execution path is corrupted due to the presence of the double quotation mark character. The workaround is to load the YAML configuration in Policy Studio, rename the offending filter by removing the quotation marks, and redeploy the configuration.

Note that this issue does not affect runtime filter execution nor when using an XML configuration.

Related Issue: RDAPI-28388

### YAML validation is failing on Windows only

Running the `yamles validate` command against an API Manager YAML configuration causes the validation step to fail. Example command:

```
yamles validate --source C:\Temp\RDAPI-22396\yaml
```

The issue occurs for API Manager configurations generated both from an XML FED (via `yamles fed2yaml`) or created via Policy Studio.

As a workaround, you can bypass the validation issue by adding `yaml:file/` before the directory path:

```
yamles validate --source yaml:file:/C:\Temp\RDAPI-22396\yaml 
```

Related issue: RDAPI-28825

## Documentation

To find all available documentation for this product version:

1. Go to [Manuals on the Axway Documentation portal](https://docs.axway.com/bundle).
2. In the left pane **Filters** list, select your product or product version.

Customers with active support contracts must log in to access restricted content.

For information on the different operating systems, databases, browsers, and thick client platforms supported by each Axway product, see [Supported Platforms](https://docs.axway.com/bundle/Axway_Products_SupportedPlatforms_allOS_en).

## Support services

The Axway Global Support team provides worldwide 24 x 7 support for customers with active support agreements.

Email [support@axway.com](mailto:support@axway.com) or visit [Axway Support](https://support.axway.com/).

See [Get help with API Gateway](/docs/apim_administration/apigtw_admin/trblshoot_get_help/) for the information that you should be prepared to provide when you contact Axway Support.

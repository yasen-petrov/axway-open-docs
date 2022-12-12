---
title: API Portal 7.7 November 2022 Release Notes
linkTitle: API Portal November 2022
draft: false
weight: 95
date: 2022-09-14
description: null
---
API Portal updates are cumulative, comprising new features and changes delivered in previous updates unless specifically indicated otherwise in the Release notes.

We are delighted to deliver a new user interface (UI) and custom page builder in this update. This gives customers the ability to offer a more modern look and feel to the UI and it makes it easier to customize and make significant changes to their portal's interface.

## Installation

API Portal is available as a software installation or a virtualized deployment in a Docker container. For more information, see the following options:

* If you are installing API Portal for the first time using this update, see [Install API Portal](/docs/apim_installation/apiportal_install/).
* If you are already using API Portal (7.5.x, 7.6.x, 7.7.x) and want to install this update, see [Upgrade API Portal](/docs/apim_installation/apiportal_install/upgrade_automatic/).

### Docker containers

* To deploy API Portal in Docker containers, see [Deploy API Portal in containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).
* If you are already using API Portal in Docker containers and you want to install this update, see [Upgrade API Portal in Docker containers](/docs/apim_installation/apiportal_docker/docker_portal_upgrade/).

## New features and enhancements

The following new features and enhancements are available in this update.

### New T4 template and Page Builder

In this release, API Portal ships with a brand new [T4 template](https://www.joomlart.com/t4-framework), which offers a modern developer portal user experience and enhanced options for easier customization and extendibility so that the portal can be easily adapted to match the latest branding and marketing guidelines.

The T4 template is fully customizable and extendable by using the new [T4 Page Builder](https://www.joomlart.com/t4-builder/t4-joomla-page-builder) component, which is a visual page editor that allows pages to be easily customized without coding knowledge. This includes page layouts, adding additional page content, custom modules and styling.

The template is also fully integrated with [Bootstrap 4](https://getbootstrap.com/), which provides access to all of its features and components.

## Limitations of this update

This update has the following limitations:

* API Portal 7.7.20221130 is compatible with API Gateway and API Manager 7.7.20221130 only.
* To update to API Portal November 2022 (which is based on Joomla 4) from versions older than February 2022 we recommend you to upgrade to API Portal February 2022 first, then follow the [Upgrade API Portal May 2022 or latest](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-to-api-portal-may-2022-or-latest-releases) section.
* Direct update to API Portal November 2022 can be done only from API Portal May 2022 and latest by following the [Upgrade from the Joomla! Administrator Interface](/docs/apim_installation/apiportal_install/upgrade_automatic/#upgrade-from-the-joomla-administrator-interface) section.
* This update is not available as a virtual appliance or as a managed service on Axway Cloud.

## Important changes

It is important, especially when upgrading from an earlier version, to be aware of the following changes in the behavior or operation of the product in this update.

### New User Interface (T4 template)

API Portal November 22 update ships with a new UI based on the [T4 template](https://www.joomlart.com/t4-framework), and API Portal now utilizes the T4 PageBuilder - a tool that makes the customizations of page contents easier.

When upgrading from previous API Portal versions, you must [manually configure the menu items](/docs/apim_administration/apiportal_admin/customize_getting_started/#enable-the-t4-template) and apply your [customizations and styling to the new T4 template](/docs/apim_administration/apiportal_admin/customize_getting_started/#customize-using-t4-template) if you wish to take advantage of this new functionality. If you skip this manual configuration, you will be still using the T3 template.

## Deprecated features

As part of our software development life cycle, we constantly review our API Portal offering. As part of this update, the following capabilities have been deprecated.

### Purity III Template

In this release, the Purity III template is deprecated from use. Users upgrading from previous versions of API Portal will be still able to use the Purity III template and can later migrate to the new [T4 template](https://www.joomlart.com/t4-framework).

## End of support notices

There are no end of support notices in this update

## Removed features

To stay current and align our offerings with customer demand and best practices, Axway might discontinue support for some capabilities. As part of this review, the following features have been removed.

### API Portal Docker images

The API Portal Docker images are no longer available in a downloadable zip file. This was mainly provided for legacy reasons. All API Portal Docker images can be pulled directly from the [Axway repository](https://repository.axway.com/) using standard Docker commands.

## Fixed issues

This version of API Portal includes:

* Fixes from all 7.7 updates released prior to this version. For details of all the update fixes included, see the corresponding [release note](/docs/apim_relnotes/) for each 7.7 update.
* Additional fixes might be delivered as patches up to 15 months after the release date. You can find the list of patches available on top of this update on [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/20220830/ipp/100/product/545/version/3036/subtype/8). If no patches were created for this release, the link will return "No search results found".

### Fixed security vulnerabilities

| **Internal ID** | **Case ID** | **CVE Identifier** | **Description**                                                                                                                     |
| ----------- | ------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| IAP-5640    |         |                | **Issue**: Vulnerable version 2.0.0 of markedjs was used. **Resolution**: Markedjs was updated to version 4.0.19, which is not vulnerable. |

### Other fixed issues

| **Internal ID** | **Case ID**                    | **Description**                                                                                                                                                                                                                               |
| ----------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| IAP-4344    | 1261063                      | **Issue**: Markdown output was centered in API Details page. **Resolution**: Markdown was parsed correctly in API Catalog and API Details pages.                                                                                                   |
| IAP-5084    | 01343090, 01384715           | **Issue**: No way to disable Session hijack prevention feature. **Resolution**: Added Session hijack prevention feature by way of enable/disable toggle in JAI, and with environment variable for docker.                                                     |
| IAP-5516    | 01395869, 01325865, 01407024 | **Issue**: ThemeMagic button is no longer available in Joomla 4 (After API Portal February 2022 versions). **Resolution**: Documentation was updated to customize with manual steps instead of ThemeMagic functionality.                                 |
| IAP-5618    | 1402723                      | **Issue**: Enabling Redis cache in API Portal container through environment variable or in software installation was not working due to Joomla configuration changes. **Resolution**: New Joomla 4.2.x configurations are now consider in API Portal. |
| IAP-5619    | 1406017                      | **Issue**: API Portal is throwing `mime\_type` error when accessing media content in JAI “Media > Content” due to new PHP dependency requirements in Joomla 4.2.x. **Resolution**: Installing php80-fileinfo-8.0.24 dependency.                       |

## Known issues

The following are known issues for this update.

### Data Request feature is not available for SSO users without email in New UI page builder design

[Data Request](/docs/apim_administration/apiportal_admin/manage_privacy_personal_data/#manage-data-requests) feature is not able to send request for SSO users without email only for New UI page builder design - T4 and Page Builder. The functionality is available, but the emails are automatically detected from Joomla login details, while for SSO users registered without emails this information is not available and the request generation is not able to proceed. For users upgrading from versions earlier than November 22 and still using T3 and Purity III, the stylization for API Portal Data Request functionality works as before.

Related Issue: IAP-5649

### Two-factor authentication is not available with Joomla 4.2.x in API Portal

Joomla 4.2.0 includes changes around two(multi)-factor authentications. However, this functionality is not yet available in API Portal. For more information, see [Joomla 4.2 release notes](https://www.joomla.org/announcements/release-news/5865-joomla-4-2-release.html).

Related issue: IAP-5504

### Custom properties permissions per role are not respected in some scenarios

Custom properties permissions per role will not be respected for the organization in the following scenarios:

* Permissions will not be respected for the organization the API is assigned to because respecting them will cause significant performance impact on the Try-it page. The custom properties permissions will follow the role of the user's primary organization.
* On creating an application, the App has no organization yet, so the custom properties permissions cannot be respected per role of the organization. They will follow the permissions for the role of the primary organization of the user.

Related issue: IAP-4474

### Page layout and alignment for Arabic language

Changing the API Portal language to Arabic (or any other right to left language) results in issues with page layout and alignment on the API Portal Home and Pricing pages, and some buttons are not visible. As a workaround, you can turn on the development mode in JAI. Follow these steps:

1. Log in to Joomla! Admin Interface (JAI).
2. In the JAI top navigation bar, click **Extensions > Templates**.
3. Click your template style (for example, `purity_III * Default`) to open it.
4. Click the **General** tab.
5. Change **Development Mode** to `ON`.
6. Click **Save** and click **Close** to close the template style.

Related issue: IAP-308

## Documentation

To find all available documentation for this product version:

1. Go to [Manuals on the Axway Documentation portal](https://docs.axway.com/bundle).
2. In the left pane **Filters** list, select your product or product version.

Customers with active support contracts need to log in to access restricted content.

For information on the different operating systems, databases, browsers, and thick client platforms supported by each Axway product, see [Supported Platforms](https://docs.axway.com/bundle/Axway_Products_SupportedPlatforms_allOS_en).

## Support services

The Axway Global Support team provides worldwide 24 x 7 support for customers with active support agreements.

Email [support@axway.com](mailto:support@axway.com) or visit [Axway Support](https://support.axway.com/).

See [Get help with API Gateway](/docs/apim_administration/apigtw_admin/trblshoot_get_help/) for the information that you should be prepared to provide when you contact Axway Support.

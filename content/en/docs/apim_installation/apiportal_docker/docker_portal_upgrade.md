{
"title": "Upgrade API Portal container deployment",
  "linkTitle": "Upgrade container deployment",
  "weight": "30",
  "date": "2021-01-05",
  "description": "Upgrade your API Portal container deployment to a newer version."
}

## General upgrade

To upgrade your API Portal container deployment, perform the following:

1. Always back up your database and data from volumes before an upgrade.
2. Obtain a newer API Portal Docker image, available from [Axway Repository](https://repository.axway.com).
3. Remove the running container:

    ```
    docker container rm -f <old-container-name>
    ```

4. Recreate a new container with the same docker run parameters as the old one:

    ```
    docker container run <old-parameters> <newer-image>
    ```

See more [Run a Docker container using the image](/docs/apim_installation/apiportal_docker/docker_portal_run_image/#run-a-docker-container-using-the-image).

The upgrade preserves any API Portal customizations stored in volumes or database.

{{% alert title="Note" %}}
There is no support for downgrading an API Portal container deployment. Running an older API Portal container against a newer database schema will result in failure.
{{% /alert %}}

## Upgrade from versions before May 2022 to latest release

API Portal versions after [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/) are integrated with *Joomla 4*, which results in some backward incompatible changes. Attempting to upgrade directly from versions prior to [May 2022](/docs/apim_relnotes/20220530_apip_relnotes/) to latest release will break database integrity.

To upgrade API Portal from versions higher than February 2022 to the latest release, follow the [General upgrade](#general-upgrade) section.

To upgrade from before May 2022 to API Portal latest release, follow these steps:

1. If your API Portal version is lower than [February 2022](/docs/apim_relnotes/20220228_apip_relnotes/), you must first upgrade to February 2022 as described in the [General upgrade](#general-upgrade) section.
2. After you installation is updated to the February 2022 release, stop the February 2022 container and start a new container from the latest image with the old parameters.

    {{% alert title="Note" %}}It is important that you use plain MySQL connection to ensure the `MYSQL_SSL_ON` environment variable is set to `0`, and `MYSQL_USER` and `MYSQL_PASSWORD` environment variables contain credentials for plain connection. For more information on how to create a plain MySQL account, see [Create a MySQL user account without TLS authentication](/docs/apim_installation/apiportal_install/install_software_configure_database/#configure-a-user-account-without-authentication).{{% /alert %}}

    ```shell
    docker container rm -f <feb22-container-name>
    docker container run \
      <old-parameters-from-prior-to-latest-container> <latest-image>
    ```
3. (Optional) If you have used MySQL SSL connection to start your new container, you can now change its value back to what was there before.

## Apply a service pack or patch on a container deployment

Service packs and patches provide important security updates, fixes for known issues, and improved performance for API Portal and its components.

To apply a service pack or patch to a container deployment:

1. Download a new API Portal Docker image that includes the service pack or patch from [Axway Repository](https://repository.axway.com).
2. Follow the steps in [Upgrade API Portal in Docker containers](/docs/apim_installation/apiportal_docker/upgrade_docker/).

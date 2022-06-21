{
"title": "Upgrade API Portal container deployment",
  "linkTitle": "Upgrade container deployment",
  "weight": "30",
  "date": "2021-01-05",
  "description": "Upgrade your API Portal container deployment to a newer version."
}

To upgrade your API Portal container deployment, perform the following:

1. Always backup your database and data from volumes before an upgrade.
2. Obtain a newer API Portal Docker image, available from [Axway Support](https://support.axway.com/en/search/index/type/Downloads/q/API%20Portal%20/ipp/10/product/545/version/3036/subtype/89).
3. Remove the running container:

    ```
    docker container rm -f <old-container-name>
    ```

4. Recreate a new container with the same docker run parameters as the old one:

    ```
    docker container run <old-parameters> <newer-image>
    ```

The upgrade preserves any API Portal customizations stored in volumes or database.

{{% alert title="Note" %}}
There is no support for downgrading an API Portal container deployment. Running an older API Portal container against a newer database schema will result in failure.
{{% /alert %}}

## Upgrade to May2022 release

With May2022 release (7.7.20220530) API&nbsp;Portal introduces Joomla!&nbsp;4 integration which brings some backward incompatible changes. Direct upgrade with the steps from the above section will break database integrity. 

In order to upgrade to May2022 please follow the steps below:

1. Go to [Axway Support Site](https://support.axway.com) and download `APIPortal_7.7.20220530_ThirdPartyPackages.zip` package.
2. If your API&nbsp;Portal container version is below Feb2022 upgrade it to Feb2022 release in a normal way as described in the above section.
3. Create an .ini file with the following content:

    ```ini
    post_max_size = 60m
    upload_max_filesize = 60m
    ```
   
    You can name this file however you like. For our example we'll name it `apiportal.ini`

4. Restart API&nbsp;Portal container with the file from the previous step mounted to the container's php configuration directory with the commands:

    ```shell
    docker container rm -f <feb22-container-name>
    docker container run \
      -v <path-to-apiportal.ini-file>:/etc/php7/conf.d/apiportal.ini \
      <old-parameters> -e MYSQL_SSL_ON=0 <feb22-image>
    ```
    
    It is important for steps 4 - 6 to use plain MySQL connection. So you must make sure `MYSQL_SSL_ON` environment variable is set to `0` and `MYSQL_USER` and `MYSQL_PASSWORD` environment variables contain credentials for plain connection. You may need to create a MySQL user without authentication, see instructions [here](/docs/apim_installation/apiportal_install/install_software_configure_database/#configure-a-user-account-without-authentication).
5. With a newly started container open API&nbsp;Portal in a browser and go to **JAI > Extensions > Plugins**, search and disable *T3&nbsp;Framework* plugin.
6. Go to **JAI > Components > Joomla!&nbsp;Update > Upload&nbsp;&&nbsp;Update**, and apply Joomla!&nbsp;4 upgrade package via uploading Joomla! upgrade package from `APIPortal_7.7.20220530_ThirdPartyPackages.zip` archive. Wait for the upgrade process to finish and login back to JAI.
7. Go to **JAI > System > Install > Extensions > Upload&nbsp;Package&nbsp;File**, and apply T3 System Plugin from `APIPortal_7.7.20220530_ThirdPartyPackages.zip` archive.
8. Stop the Feb22 container and start a new one from May22 image with the old parameters, but without mounted `apiportal.ini` file. If you used MySQL SSL connection, now you can return it back.

    ```shell
    docker container rm -f <feb22-container-name>
    docker container run \
      <old-parameters-from-prior-to-may22-container> <may22-image>
    ```

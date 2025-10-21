# Azure Blob storage

!!! admonition "Version added: [1.5.0](../release-notes/1.5.0.md)"

Companies with Microsoft-based infrastructure can set up Percona Backup for MongoDB with less administrative efforts by using [Microsoft Azure Blob Storage :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) as the remote backup storage.
 
## Create a blob storage

You can create a blob storage either via the Azure Portal web interface or using the Azure CLI.

For either method you need a storage account.

=== "Azure Portal"

    1. Sign in to Azure Portal
    2. On the Home page, select **Storage accounts**.
    3. Click **Create** from the toolbar and use the wizard to create a storage account.
    4. After the storage account is created, select it from the list.
    5. In the left menu, select **Data storage** -> **Containers**.
    6. Click **+ Container** to create a new container.
    7. Enter a name for the container and select **Private** as the access level.
    8. Click **Create** to create the container.

=== "Azure CLI"

    1. Install the [Azure CLI :octicons-link-external-16:](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). After the installation, the `az` is available for you.
    2. Sign in to Azure CLI:

        ```{.bash data-prompt="$"}
        $ az login
        ```

    3. Create a Resource group if it's not created for you:

        ```{.bash data-prompt="$"}
        $ az group create --name <your-resource-group> --location <your-location>
        ```

        For the list of available locations, run:

        ```{.bash data-prompt="$"}
        $ az account list-locations
        ```

    4. Create a storage account:

        ```{.bash data-prompt="$"}
        $ az storage account create --name <storage-account-name> --resource-group <your-resource-group> --location <your-location> --sku Standard_LRS
        ```

    4. Create a blob container:

        ```{.bash data-prompt="$"}
        $ az storage container create --account-name <storage-account-name> --name <your-container>  --public-access off
        ```

        ??? example "Expected output"

            ```{.json .no-copy}
            {
              "created": true
            }
            ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

## Configuration example

You can find [the configuration file template :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

```yaml
storage:
  type: azure
  azure:
    account: <your-storage-account>
    container: <your-container>
    prefix: pbm
    credentials:
      key: <your-access-key>
```

For the description of configuration options, see [Configuration file options](../reference/configuration-options.md).


## Upload retries 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to Microsoft Azure storage as well as the min and max time to wait for the next retry. Set the options `storage.azure.retryer.numMaxRetries`, `storage.azure.retryer.minRetryDelay` and `storage.azure.retryer.maxRetryDelay` in Percona Backup for MongoDB configuration.

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 800
  maxRetryDelay: 60
```

This upload retry increases the chances of data upload completion in cases of unstable connection.
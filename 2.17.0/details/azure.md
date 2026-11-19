> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Azure Blob storage

Version added: [1.5.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/1.5.0.html)

Companies with Microsoft-based infrastructure can set up Percona Backup for MongoDB with less administrative efforts by using [Microsoft Azure Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) as the remote backup storage.

## Create a blob storage

You can create a blob storage either via the Azure Portal web interface or using the Azure CLI.

For either method you need a storage account.

**Azure Portal**

1. Sign in to Azure Portal
1. On the Home page, select **Storage accounts**.
1. Click **Create** from the toolbar and use the wizard to create a storage account.
1. After the storage account is created, select it from the list.
1. In the left menu, select **Data storage** -> **Containers**.
1. Click **+ Container** to create a new container.
1. Enter a name for the container and select **Private** as the access level.
1. Click **Create** to create the container.

**Azure CLI**

1. Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) . After the installation, the `az` is available for you.

1. Sign in to Azure CLI:

   ```
   az login
   ```

1. Create a Resource group if it’s not created for you:

   ```
   az group create --name <your-resource-group> --location <your-location>
   ```

   For the list of available locations, run:

   ```
   az account list-locations
   ```

1. Create a storage account:

   ```
   az storage account create --name <storage-account-name> --resource-group <your-resource-group> --location <your-location> --sku Standard_LRS
   ```

1. Create a blob container:

   ```
   az storage container create --account-name <storage-account-name> --name <your-container>  --public-access off
   ```

   Expected output

   ```
   {
     "created": true
   }
   ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/storage-configuration.html#permissions-setup).

## Configuration example

You can find [the configuration file template](https://github.com/percona/percona-backup-mongodb/blob/v2.17.0/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

```
storage:
  type: azure
  azure:
    account: <your-storage-account>
    container: <your-container>
    prefix: pbm
    credentials:
      key: <your-access-key>
```

For the description of configuration options, see [Configuration file options](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/configuration-options.html).

## Upload retries

You can set up the number of attempts for Percona Backup for MongoDB to upload data to Microsoft Azure storage as well as the min and max time to wait for the next retry. Set the options `storage.azure.retryer.numMaxRetries`, `storage.azure.retryer.minRetryDelay` and `storage.azure.retryer.maxRetryDelay` in Percona Backup for MongoDB configuration.

```
retryer:
  numMaxRetries: 3
  minRetryDelay: 800ms
  maxRetryDelay: 60s
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

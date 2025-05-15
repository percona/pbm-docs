# Azure Blob storage

!!! admonition "Version added: [1.5.0](../release-notes/1.5.0.md)"

Companies with Microsoft-based infrastructure can set up Percona Backup for MongoDB with less administrative efforts by using [Microsoft Azure Blob Storage :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) as the remote backup storage.
 

## Configuration example

```yaml
storage:
  type: azure
  azure:
    account: <your-account>
    container: <your-container>
    prefix: pbm
    credentials:
      key: <your-access-key>
```

For the description of configuration options, see [Configuration file options](../reference/configuration-options.md).

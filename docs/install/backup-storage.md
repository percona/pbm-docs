# Configure remote backup storage

The easiest way to provide remote backup storage configuration is to specify it in a YAML config file and upload this file to Percona Backup for MongoDB using `pbm` CLI.

The storage configuration itself is out of scope of the present document. We assume that you have configured one of the supported remote backup storages.
{.power-number}

1. Create a config file (e.g. `pbm_config.yaml`).

2. Specify the storage information within.

    The following is the sample configuration for Amazon AWS:

    ```yaml
    storage:
      type: s3
      s3:
        region: us-west-2
        bucket: pbm-test-bucket
        prefix: data/pbm/backup
        credentials:
          access-key-id: <your-access-key-id-here>
          secret-access-key: <your-secret-key-here>
        serverSideEncryption:
          sseAlgorithm: aws:kms
          kmsKeyID: <your-kms-key-here>
    ```

    This is the sample configuration for Microsoft Azure Blob storage:

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

    This is the sample configuration for filesystem storage:

    ```yaml
    storage:
      type: filesystem
      filesystem:
        path: /data/local_backups
    ```

    See more examples in [Configuration file examples](../details/storage-config-example.md).


3. Insert the config file

```{.bash data-prompt="$"}
$ pbm config --file pbm_config.yaml
```

To learn more about Percona Backup for MongoDB configuration, see [Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](../reference/config.md).

## Next steps

[Start `pbm-agent` :material-arrow-right:](start-pbm-agent.md){.md-button}
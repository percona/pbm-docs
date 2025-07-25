# Configure remote backup storage

The easiest way to provide remote backup storage configuration is to specify it in a YAML config file and upload this file to Percona Backup for MongoDB using `pbm` CLI.

The storage configuration itself is out of scope of the present document. We assume that you have configured one of the supported remote backup storages and provisioned access keys with the proper permissions for PBM. See [Remote Backup Storage](../details/storage-configuration.md) for more details.

## Considerations

Percona Backup for MongoDB needs its own dedicated S3 bucket exclusively for backup-related files. Ensure that this bucket is created and managed solely by PBM.

## Procedure {.power-number}

1. Create a config file (e.g. `pbm_config.yaml`). You can use the [template configuration file :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) and modify it as needed.

    === ":material-aws: Amazon AWS"    

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
        ```    

    === ":material-google-cloud: Google Cloud Storage"    

        ```yaml
        storage:
          type: gcs
          gcs:
             bucket: pbm-testing
             chunkSize: 16777216
             prefix: pbm/test
             credentials:
               clientEmail: <your-google-cloud-project-id-here>
               privateKey: <your-private-key-here>
        ```    

    === ":material-microsoft-azure: Microsoft Azure Blob Storage"    

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

    === ":material-file-tree: Shared local filesystem"    

        ```yaml
        storage:
          type: filesystem
          filesystem:
            path: /data/local_backups
        ```    

    Navigate to every storage page for a detailed example configuration file.

2. Apply the config file to PBM

    ```{.bash data-prompt="$"}
    $ pbm config --file pbm_config.yaml
    ```

To learn more about Percona Backup for MongoDB configuration, see [Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](../reference/config.md).

## Next steps

[Start `pbm-agent` :material-arrow-right:](start-pbm-agent.md){.md-button}

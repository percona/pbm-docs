# Configure remote backup storage

The easiest way to provide remote backup storage configuration is to specify it in a YAML config file and upload this file to Percona Backup for MongoDB using `pbm` CLI.

The storage configuration itself is out of scope of the present document. We assume that you have configured one of the supported remote backup storages and provisioned access keys with the proper permissions for PBM. See [Remote Backup Storage](../details/storage-configuration.md) for more details.

!!! Info 

    Percona Backup for MongoDB needs its own dedicated S3 bucket exclusively for backup-related files. Ensure that this bucket is created and managed solely by PBM.
    
{.power-number}

1. Create a config file (e.g. `pbm_config.yaml`).

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
         type: s3
             s3:
             region: northamerica-northeast1
             bucket: pbm-testing
             prefix: pbm/test
             endpointUrl: https://storage.googleapis.com
             credentials:
               access-key-id: <your-access-key-id-here>
               secret-access-key: <your-secret-key-here>
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

    === ":material-file-tree: Shared Local Filesystem"    

        ```yaml
        storage:
          type: filesystem
          filesystem:
            path: /data/local_backups
        ```    

    See more examples in [Configuration file examples](../details/storage-config-example.md).

2. Apply the config file to PBM

    ```{.bash data-prompt="$"}
    $ pbm config --file pbm_config.yaml
    ```

To learn more about Percona Backup for MongoDB configuration, see [Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](../reference/config.md).

## Next steps

[Start `pbm-agent` :material-arrow-right:](start-pbm-agent.md){.md-button}

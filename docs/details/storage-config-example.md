# Remote backup storage configuration examples

Provide the remote backup storage configuration as a YAML config file. The following are the examples of config files for [supported remote storages](storage-configuration.md#supported-storage-types). For how to insert the config file, see [Insert the whole Percona Backup for MongoDB config from a YAML file](../reference/config.md).

## S3-compatible remote storage

### Amazon Simple Storage Service

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

### GCS

```yaml
storage:
 type: s3
 s3:
     region: us-east1
     bucket: pbm-testing
     prefix: pbm/test
     endpointUrl: https://storage.googleapis.com
     credentials:
       access-key-id: <your-access-key-id-here>
       secret-access-key: <your-secret-key-here>
```

### MinIO

```yaml
storage:
  type: s3
  s3:
    endpointUrl: "http://localhost:9000"
    region: my-region
    bucket: pbm-example
    prefix: data/pbm/test
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
```

## Remote filesystem server storage

```yaml
storage:
  type: filesystem
  filesystem:
    path: /data/local_backups
```

## Microsoft Azure Blob Storage

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

> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Configure remote backup storage

The easiest way to provide remote backup storage configuration is to specify it in a YAML config file and upload this file to Percona Backup for MongoDB using `pbm` CLI.

The storage configuration itself is out of scope of the present document. We assume that you have configured one of the supported remote backup storages and provisioned access keys with the proper permissions for PBM. See [Remote Backup Storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/storage-configuration.html) for more details.

## Considerations

Percona Backup for MongoDB needs its own dedicated S3 bucket exclusively for backup-related files. Ensure that this bucket is created and managed solely by PBM.

## Procedure

1. Create a config file (e.g. `pbm_config.yaml`). You can use the [template configuration file](https://github.com/percona/percona-backup-mongodb/blob/v2.17.0/packaging/conf/pbm-conf-reference.yml) and modify it as needed.

   **Amazon AWS**

   ```
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

   **S3-compatible (MinIO)**

   ```
   storage:
     type: minio
     minio:
       endpoint: minio.example.com:9000
       secure: true
       bucket: pbm-test-bucket
       prefix: data/pbm/backup
       credentials:
         access-key-id: <your-access-key-id-here>
         secret-access-key: <your-secret-key-here>
   ```

   **GCS (service account)**

   ```
   storage:
     type: gcs
     gcs:
        bucket: pbm-testing
        chunkSize: 16777216
        prefix: pbm/test
        credentials:
          clientEmail: <your-service-account-email>
          privateKey: <your-service-account-private-key-here>
   ```

   **Microsoft Azure Blob Storage**

   ```
   storage:
     type: azure
     azure:
       account: <your-account>
       container: <your-container>
       prefix: pbm
       credentials:
         key: <your-access-key>
   ```

   **Alibaba Cloud Storage**

   ```
   storage:
   type: oss
   oss:
     region: eu-central-1
     bucket: your-bucket-name
     endpointUrl: https://oss-eu-central-1.aliyuncs.com
     credentials:
       accessKeyID: "STS.****************"
       accessKeySecret:  "3dZn*******************************************"
   ```

   **Shared local filesystem**

   ```
   storage:
     type: filesystem
     filesystem:
       path: /data/local_backups
   ```

   Navigate to every storage page for a detailed example configuration file.

1. Apply the config file to PBM

   ```
   pbm config --file pbm_config.yaml
   ```

To learn more about Percona Backup for MongoDB configuration, see [Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/config.html).

## Next steps

[Start `pbm-agent`](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/start-pbm-agent.html)

# Manage large backup files upload 

!!! admonition "Version added: [2.11.0](../release-notes/2.11.0.md)

As your database grows, so do your backups. Eventually, a collection or index  may become so large that its backup file exceeds the maximum object size limit of your cloud or local storage. When this happens, Percona Backup for MongoDB (PBM) can't upload the file, which can disrupt your backup strategy.

The following table provides default maximum size limits for the supported backup storages:

| Storage | Default size limit|
| :--- | :--- |
| **AWS S3** | 4.9 TB |
| **GCS** | 4.9 TB |
| **Azure Blob Storage** | 190 TB |
| **Filesystem storage** | 4.9 TB |

These defaults are sufficient to satisfy the majority of use cases. However, you can configure a new maximum size for backup files for the storage you use. To do this, define the file size in GB for the `maxObjSizeGB` configuration parameter. 

Define new limits with caution, only when you are absolutely sure in your actions. Setting a too low value may cause a single backup file to be split into a too large number of smaller files and this may affect performance.

This is the configuration example for the AWS S3 storage:

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
  maxObjSizeGB: 5018
```

## How it works

When a backup file exceeds the configured size limit, PBM does the following:

* Splits the file into pieces, each of which doesn't exceed the defined size limit. PBM respects your compression settings, so compressed backups stay compressed after the split
* Names each piece by adding the `pbmpart{number}` token to the filename. This lets PBM identify and manage all the pieces of a single backup.
* Uploads the pieces to the storage

When reading data for an operation like a restore, PBM automatically reverses this process: it retrieves all the pieces from storage, merges them back into a single file, and proceeds with the command. This makes the entire process transparent to your application.

With this ability to set the maximum file size, you can future-proof your backup strategy and gain peace of mind knowing your data is always safe and recoverable. 
> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Remote backup storage

Backup storage is a critical component of any database backup strategy. It serves as a secure, reliable place for your MongoDB data backups, ensuring that your data is protected and can be recovered when needed. The choice of backup storage directly impacts your backup strategy’s reliability, performance, and cost-effectiveness.

The backup storage serves several purposes:

- Provides a secure location for storing backup data
- Ensures data durability and availability
- Allows for backup data portability across different environments

## Supported storage types

Percona Backup for MongoDB supports the following storage types:

- [Amazon S3](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/s3-storage.html)
- [Google Cloud storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/gcs.html)
- [MinIO and S3-compatible storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/minio.html)
- [Filesystem server storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/filesystem-storage.html)
- [Microsoft Azure Blob storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/azure.html)
- [Oracle Cloud Infrastructure Object Storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/oci-storage.html)
- [Alibaba Cloud OSS storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/oss.html)

### Considerations for choosing a backup storage type for s3-compatible storages

When configuring Percona Backup for MongoDB, choose the storage type based on your storage service:

- **AWS S3**: Use the `s3` storage type for native Amazon S3 storage.
- **S3-compatible storage**: For S3-compatible storage services that support Signature Version 4 (SigV4) used in AWS SDK v2, use the `s3` storage type. If you encounter upload or download errors in the PBM logs, switch to the `minio` storage type.
- **Google Cloud Storage**: Always use the dedicated `gcs` storage type. See [Google Cloud Storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/gcs.html) for configuration details.

## How PBM organizes backups on the storage

Percona Backup for MongoDB (PBM) saves backup data to a designated directory on the backup storage. It can be a specific directory you define for the storage or the root folder.

Each backup is prefixed with the UTC starting time for easy identification and consists of:

- A metadata file containing backup information
- For each replica set:
- A compressed mongodump archive of all collections
- A compressed BSON file containing the oplog entries for the backup period

The oplog entries ensure backup consistency, and the end time of the oplog slice(s) is the data-consistent point in time of a backup snapshot.

Using the [`pbm list`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-list) or [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) commands, you can scan the backup directory to find existing backups, even if you never used PBM on your computer before.

## Permissions setup

Regardless of the remote backup storage you use, grant the `List/Get/Put/Delete` permissions to this storage for the user identified by the access credentials.

The following example shows the permissions configuration to the `pbm-testing` bucket on the AWS S3 storage.

```
{
    "Version": "2021-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::pbm-testing"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::pbm-testing/*"
        }
    ]
}
```

### Storage-specific documentation

Please refer to the documentation of your selected storage for the data access management.

See also

- AWS documentation: [Controlling access to a bucket with user policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)
- Google Cloud Storage documentation: [Overview of access control](https://cloud.google.com/storage/docs/access-control)
- Microsoft Azure documentation: [Assign an Azure role for access to blob data](https://docs.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access?tabs=portal)
- MinIO documentation: [Policy Management](https://docs.min.io/minio/baremetal/security/minio-identity-management/policy-based-access-control.html)
- Alibaba Cloud documentation: [Permissions and access control](https://www.alibabacloud.com/help/en/oss/user-guide/permissions-and-access-control)

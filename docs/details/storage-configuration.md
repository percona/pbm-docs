# Remote backup storage

Backup storage is a critical component of any database backup strategy. It serves as a secure, reliable place for your MongoDB data backups, ensuring that your data is protected and can be recovered when needed. The choice of backup storage directly impacts your backup strategy's reliability, performance, and cost-effectiveness.

The backup storage serves several purposes:

* Provides a secure location for storing backup data
* Ensures data durability and availability
* Allows for backup data portability across different environments

Percona Backup for MongoDB (PBM) saves backup data to a designated directory on the backup storage. It can be a specific directory you define for the storage or the root folder. Each backup is prefixed with the UTC starting time for easy identification and consists of:

* A metadata file containing backup information
* For each replica set:

   - A compressed mongodump archive of all collections
   - A compressed BSON file containing the oplog entries for the backup period

The oplog entries ensure backup consistency, and the end time of the oplog slice(s) is the data-consistent point in time of a backup snapshot.

Using the [`pbm list`](../reference/pbm-commands.md#pbm-list) or [`pbm status`](../reference/pbm-commands.md#pbm-status) commands, you can scan the backup directory to find existing backups, even if never used PBM on your computer before.

## Supported storage types

Percona Backup for MongoDB supports the following storage types:

* [S3-compatible storage](s3-storage.md)
* [Filesystem server storage](filesystem-storage.md)
* [Microsoft Azure Blob storage](azure.md)


## Permissions setup

Regardless of the remote backup storage you use, grant the `List/Get/Put/Delete` permissions to this storage for the user identified by the access credentials.

The following example shows the permissions configuration to the `pbm-testing` bucket on the AWS S3 storage.

```json
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

Please refer to the documentation of your selected storage for the data access management.

!!! admonition "See also"

    * AWS documentation: [Controlling access to a bucket with user policies :octicons-link-external-16:](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)
    * Google Cloud Storage documentation: [Overview of access control :octicons-link-external-16:](https://cloud.google.com/storage/docs/access-control)
    * Microsoft Azure documentation: [Assign an Azure role for access to blob data :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access?tabs=portal)
    * MinIO documentation: [Policy Management :octicons-link-external-16:](https://docs.min.io/minio/baremetal/security/minio-identity-management/policy-based-access-control.html)

*[AWS KMS]: Amazon Web Services Key Management Service

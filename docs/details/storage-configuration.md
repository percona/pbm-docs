# Remote backup storage

Percona Backup for MongoDB saves your files to a directory. Using [`pbm list`](../reference/pbm-commands.md#pbm-list), a user can scan this directory to find existing
backups even if they never used `pbm` on their computer before.

The files are prefixed with the (UTC) starting time of the backup. For each
backup, there is one metadata file. For each replica set, a backup includes the following:

* A mongodump-format compressed archive that is the dump of collections
* A (compressed) BSON file dump of the oplog covering the time span of the backup

The end time of the oplog slice(s) is the data-consistent point in time of a backup snapshot.

## Supported storage types

Percona Backup for MongoDB supports the following storage types:

* [S3-compatible storage](#s3-compatible-storage)

* [Filesystem type storage](#remote-filesystem-server-storage)

* [Microsoft Azure Blob storage](#microsoft-azure-blob-storage)

### S3-compatible storage

Percona Backup for MongoDB should work with other S3-compatible storages, but was only tested with the following ones:


* [Amazon Simple Storage Service :octicons-link-external-16:](https://docs.aws.amazon.com/s3/index.html)


* [Google Cloud Storage :octicons-link-external-16:](https://cloud.google.com/storage)


* [MinIO :octicons-link-external-16:](https://min.io/)

#### Storage bucket creation

Here are some examples of the steps required to create a bucket.

=== ":material-aws: Amazon S3"

    1. Install and configure [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

    2. Create an S3 bucket

       ```{.bash data-prompt="$"}
       $ aws s3api create-bucket --bucket my-s3-bucket --region us-east-1
       ```
      
    3. Verify the bucket creation

        ```{.bash data-prompt="$"}
        $ aws s3 ls
        ```
   
=== ":material-google-cloud: Google Cloud Storage"

    1. Install and configure the [gcloud CLI](https://cloud.google.com/sdk/docs/install)

    2. Create a bucket

       ```{.bash data-prompt="$"}
       $ gcloud storage buckets create my-gcs-bucket --location=US
       ```
      
    3. Verify the bucket creation

        ```{.bash data-prompt="$"}
        gcloud storage buckets list
        ```
        
=== ":simple-minio: MinIo"

    1. Install a [MinIO client :octicons-link-external-16:](https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc). After the installation, the `mc` is available for you.

    2. Configure the `mc` command line tool with a MinIO Server

        ```{.bash data-prompt="$"}
        $ mc alias set myminio http://127.0.0.1:9000 MINIO_ACCESS_KEY MINIO_SECRET_KEY
        ```
    
    3. Create a bucket

        ```{.bash data-prompt="$"}
        $ mc mb myminio/my-minio-bucket
        ```
      
    4. Verify the bucket creation

        ```{.bash data-prompt="$"}
        $ mc ls myminio
        ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](#permissions-setup).
        
#### Server-side encryption

Percona Backup for MongoDB supports [server-side encryption](../reference/glossary.md#server-side-encryption) for [S3 buckets](../reference/glossary.md#bucket) with the following encryption types:

* customer-provided keys stored in AWS KMS (SSE-KMS)
* customer-provided keys stored on the client side (SSE-C)
* Amazon S3 managed encryption keys (SSE-S3)

To learn more about each encryption type, refer to the following sections of Amazon AWS documentation:

* [Using server-side encryption with Amazon S3 managed keys (SSE-S3)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html)
* [Protecting Data Using Server-Side Encryption with CMKs Stored in AWS Key Management Service (SSE-KMS) :octicons-link-external-16:](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)
* [Protecting data using server-side encryption with customer-provided encryption keys (SSE-C) :octicons-link-external-16:](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerSideEncryptionCustomerKeys.html)

##### SSE-KMS encryption

!!! admonition "Version added: [1.3.2](../release-notes/1.3.2.md)" 

To use the SSE-KMS encryption, specify the following parameters in the Percona Backup for MongoDB configuration file: 

```yaml
serverSideEncryption:
   kmsKeyID: <kms_key_ID>
   sseAlgorithm: aws:kms
```  

##### SSE-C encryption

!!! admonition "Version added: [2.0.1](../release-notes/2.0.1.md)" 

Percona Backup for MongoDB also supports server-side encryption with customer-provided keys that are stored on the client side (SSE-C). Percona Backup for MongoDB provides the encryption keys as part of the requests to the S3 storage. The S3 storage uses them to encrypt/decrypt the data using the `AES-256` encryption algorithm. In such a way you save on subscribing to AWS KMS services and can use the server-side encryption with the S3-compatible storage of your choice.

!!! admonition ""

    SSE-C encryption should work with other S3-compatible storage types, but was only tested with the AWS and MinIO. Check the support of this functionality with your S3 storage provider.

!!! warning

    1. Enable/disable the server-side encryption only for the empty bucket. Otherwise, Percona Backup for MongoDB fails to save/retrieve objects to/from the storage properly.
    2. S3 storage doesn't manage or store the encryption key. It is your responsibility to track what key was used to encrypt what object in the bucket. If you lose the key, any request for an object without the encryption key fails and you lose the object. 

To use the SSE-C encryption, specify the following parameters in the Percona Backup for MongoDB configuration file:    

```yaml
serverSideEncryption:
  sseCustomerAlgorithm: AES256
  sseCustomerKey: <your_encryption_key>
``` 

##### SSE-S3 encryption

!!! admonition "Version added: [2.6.0](../release-notes/2.6.0.md)" 

Percona Backup for MongoDB supports server-side encryption with Amazon S3 managed keys (SSE-S3), the default encryption method in Amazon AWS. All new objects added to an S3 bucket are automatically encrypted without impacting performance.

To use SSE-S3 encryption, specify the following parameters in the Percona Backup for the MongoDB configuration file:

```yaml
serverSideEncryption:
   sseAlgorithm: AES256
```  

#### Support of multiple endpoints to the same S3 storage

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)" 

In environments where `pbm-agents` run on servers that are distributed across several data centers, accessing the same remote backup storage can become challenging. This can be due to complex network configurations or strict policies that prevent direct connections to the outside world. As a result, `pbm-agents` can't use the same URL to reach the storage, which is necessary for Percona Backup for MongoDB to work properly.

To address these challenges, you can configure custom endpoint URLs for specific nodes in the PBM configuration. This allows all `pbm-agents` to access the same storage while respecting the network settings of their data centers.

The supported storage types are Amazon S3 and Microsoft Azure Blob storage. 

Here’s the example of the configuration file with the endpoint map:

```yaml
storage:
    type: s3
    s3:
      endpointUrl: http://S3:9000
      endpointUrlMap:
        "node01:27017": "did.socf.s3.com"
        "node03:27017": "https://example.aws.s3.com"
      ...
```

You can define the specific nodes for the `endpointUrlMap` parameter. Not listed nodes use the URL defined for the `endpointUrl` parameter. 

For the solution to work, you should also have the mapping mechanism in place. This mechanism should be able to map the custom endpoint URLs to the main endpoint URL of the storage, routing the requests from `pbm-agents` to the storage and back seamlessly.

With this ability to control the endpoints for `pbm-agents` to reach the same storage, you reduce the administrative overhead on PBM configuration and ensure its proper functioning. 

#### Debug logging

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

You can enable debug logging for different types of S3 requests in Percona Backup for MongoDB. Percona Backup for MongoDB prints S3 log messages in the `pbm logs` output so that you can debug and diagnose S3 request issues or failures.

To enable S3 debug logging, set the `storage.s3.DebugLogLevel` option in Percona Backup for MongoDB configuration. The supported values are: `LogDebug`, `Signing`, `HTTPBody`, `RequestRetries`, `RequestErrors`, `EventStreamBody`.

#### Storage classes 

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

Percona Backup for MongoDB supports [Amazon S3 storage classes :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/). Knowing your data access patterns, you can set the S3 storage class in Percona Backup for MongoDB configuration. When Percona Backup for MongoDB uploads data to S3, the data is distributed to the corresponding storage class. The support of S3 bucket storage types allows you to effectively manage S3 storage space and costs.

To set the storage class, specify the `storage.s3.storageClass` option in Percona Backup for MongoDB configuration file

```yaml
storage:
  type: s3
  s3:
    storageClass: INTELLIGENT_TIERING
```

When the option is undefined, the S3 Standard storage type is used.

#### Configure upload retries 

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to S3 storage as well as the min and max time to wait for the next retry. Set the options `storage.s3.retryer.numMaxRetries`, `storage.s3.retryer.minRetryDelay` and `storage.s3.retryer.maxRetryDelay` in Percona Backup for MongoDB configuration.

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 30
  maxRetryDelay: 5
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

#### Data upload for storage with self-issued TLS certificates

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)"

Percona Backup for MongoDB supports data upload to S3-like storage that supports self-issued TLS certificates. To make this happen, disable the TLS verification of the S3 storage in Percona Backup for MongoDB configuration:

```{.bash data-prompt="$"}
$ pbm config --set storage.s3.insecureSkipTLSVerify=True
```

!!! warning 

    Use this option with caution as it might leave a hole for man-in-the-middle attacks.

### Remote filesystem server storage

This storage must be a remote file server mounted to a local directory. It is the responsibility of the server administrators to guarantee that the same remote directory is mounted at exactly the same local path on all servers in the
MongoDB cluster or non-sharded replica set.

!!! warning

    Percona Backup for MongoDB uses the directory as if it were any normal directory, and does not attempt to confirm it is mounted from a remote server.

    If the path is accidentally a normal local directory, errors will eventually
    occur, most likely during a restore attempt. This will happen because **pbm-agent** processes of other nodes in the same replica set can’t access backup archive files in a normal local directory on another server.

### Local filesystem storage

This cannot be used except if you have a single-node replica set. (See the warning note above as to why). We recommend using any object store you might be already familiar with for testing. If you don’t have an object store yet, we recommend using MinIO for testing as it has simple setup. If you plan to use a remote filesytem-type backup server, please see the [Remote Filesystem Server Storage](#remote-filesystem-server-storage) above.

### Microsoft Azure Blob storage

!!! admonition "Version added: [1.5.0](../release-notes/1.5.0.md)"

You can use [Microsoft Azure Blob Storage :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) as the remote backup storage for Percona Backup for MongoDB.

This gives users a vendor choice. Companies with Microsoft-based infrastructure can set up Percona Backup for MongoDB with less administrative efforts.

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

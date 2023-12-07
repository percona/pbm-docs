# Remote backup storage options

## Common options

### storage.type


*Type*: string <br>
*Required*:     YES   


Remote backup storage type. Supported values: `s3`, `filesystem`, `azure`.

## S3 type storage options

```yaml
storage:
  type: s3
  s3:
    region: <string>
    bucket: <string>
    prefix: <string>
    endpointUrl: <string>
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
      session-token: <string>
    uploadPartSize: <int>
    maxUploadParts: <int>
    storageClass: <string>
    serverSideEncryption:
      sseAlgorithm: aws:kms
      kmsKeyID: <your-kms-key-here>
      sseCustomerAlgorithm: AES256
      sseCustomerKey: <your_encryption_key>
    retryer:
      numMaxRetries: 3
      minRetryDelay: 30
      maxRetryDelay: 5
```

### storage.s3.provider


*Type*: string <br>
*Required*: NO

The storage provider’s name. 

Supported values: `aws`, `gcs`

### storage.s3.bucket


*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [AWS Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) and [GCS bucket naming guidelines](https://cloud.google.com/storage/docs/naming-buckets#requirements) for bucket name requirements.

### storage.s3.region

*Type*: string <br>
*Required*: YES (for AWS and GCS)

The location of the storage bucket.
Use the [AWS region list](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) and [GCS region list](https://cloud.google.com/storage/docs/locations) to define the bucket region

### storage.s3.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory on the bucket. If undefined, backups are stored in the bucket root directory

### storage.s3.endpointUrl

*Type*: string <br>
*Required*: YES (for MinIO and GCS)

The URL to access the bucket. The default value for GCS is `https://storage.googleapis.com`

### storage.s3.forcePathStyle

*Type*: boolean <br>
*Required*: NO

By default, PBM uses the [path-style URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html#path-style-access) for accessing the S3 bucket. Setting this option to `false` instructs PBM to send virtual-hosted–style requests instead.

### storage.s3.credentials.access-key-id

*Type*: string <br>
*Required*: YES

Your access key to the storage bucket. This option can be omitted when you run Percona Backup for MongoDB using an EC2 instance profile. To learn more, refer to [Automate access to S3 buckets for Percona Backup for MongoDB](../manage/automate-s3-access.md)

### storage.s3.credentials.secret-access-key

*Type*: string <br>
*Required*: YES

The key to sign your programmatic requests to the storage bucket. This option can be omitted when you run Percona Backup for MongoDB using an EC2 instance profile. To learn more, refer to [Automate access to S3 buckets for Percona Backup for MongoDB](../manage/automate-s3-access.md)

### storage.s3.credentials.session-token

*Type*: string <br>
*Required*: NO

The AWS session token used to validate the [temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) for accessing the S3 storage. 

### storage.s3.uploadPartSize

*Type*: int <br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket. Default: 10MB

Percona Backup for MongoDB automatically increases the `uploadPartSize` value if the size of the file to be uploaded exceeds the max allowed file size. (The max allowed file size is calculated with the default values of `uploadPartSize` \* [`maxUploadParts`](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#pkg-constants) and is appr. 97,6 GB).

The `uploadPartSize` value is printed in the `pbm-agent` log.

By setting this option, you can manually adjust the size of data chunks if Percona Backup for MongoDB failed to do it for some reason. The defined `uploadPartSize` value overrides the default value and is used for calculating the max allowed file size

### storage.s3.maxUploadParts

*Type*: int <br>
*Required*: NO <br>
*Default*: 10,000

The maximum number of data chunks to be uploaded to the storage bucket. Default: 10,000

By setting this option, you can override the value defined in the [AWS SDK](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#MaxUploadParts).

It can be useful when using an S3 provider that supports a smaller number of chunks for multipart uploads.

The `maxUploadParts` value is printed in the pbm-agent log.

### storage.s3.storageClass

*Type*: string <br>
*Required*: NO

The [storage class](https://aws.amazon.com/s3/storage-classes/) assigned to objects stored in the S3 bucket. If not provided, the `STANDARD` storage class will be used. This option is available in Percona Backup for MongoDB as of v1.7.0.

### storage.s3.debugLogLevels

*Type*: string <br>
*Required*: NO

Enables S3 debug logging for different types of S3 requests. S3 log messages are printed in the `pbm logs` output.

Supported values are: `LogDebug`, `Signing`, `HTTPBody`, `RequestRetries`, `RequestErrors`, `EventStreamBody`.

To specify several event types, separate them by comma. To lean more about the event types, see [the documentation](https://pkg.go.dev/github.com/aws/aws-sdk-go@v1.40.7/aws#LogLevelType)

When undefined, no S3 debug logging is performed.

### storage.s3.insecureSkipTLSVerify

*Type*: bool <br>
*Required*: NO <br>
*Default*: False

Disables the TLS verification of the S3 storage. This allows Percona Backup for MongoDB to upload data to S3-like storages that use self-issued TLS certificates. Available in Percona Backup for MongoDB as of version 1.7.0.

!!! warning 
    
    Use this option with caution as it might leave a hole for man-in-the-middle attacks.

## Server-side encryption options

### serverSideEncryption.sseAlgorithm

*Type*: string <br>
*Required*: NO 

The key management mode used for server-side encryption with the encryption keys stored in AWS KMS.

Supported value: `aws:kms`

### serverSideEncryption.kmsKeyID

*Type*: string <br>
*Required*: NO

Your customer-managed key stored in the AWS KMS.

### serverSideEncryption.sseCustomerAlgorithm

*Type*: string <br>
*Required*: NO 

The key management mode for [server-side encryption with customer-provided keys (SSE-C)](../details/storage-configuration.md#server-side-encryption).

Supported value: `AES256`

### serverSideEncryption.sseCustomerKey

*Type*: string <br>
*Required*: NO

Your custom encryption key. This key is not stored on the S3 storage side. Thus, it is your responsibility to track what data is encrypted with what key and for storing the key. 

## Upload retry options

### retryer.numMaxRetries

*Type*: int <br>
*Required*: NO <br>
*Default*: 3

The maximum number of retries to upload data to S3 storage. A zero value means no retries will be performed. Available in Percona Backup for MongoDB as of 1.7.0.

### retryer.minRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 30

The minimum time (in ms) to wait till the next retry. Available in Percona Backup for MongoDB as of 1.7.0.

### retryer.maxRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 5

The maximum time (in minutes) to wait till the next retry. Available in Percona Backup for MongoDB as of 1.7.0.

## Filesystem storage options

```yaml
storage:
  type: filesystem
  filesystem:
    path: <string>
```

### storage.filesystem.path

*Type*: string <br>
*Required*: YES

The path to the backup directory

## Microsoft Azure Blob storage options

```yaml
storage:
  type: azure
  azure:
    account: <string>
    container: <string>
    endpointUrl: <string>
    prefix: <string>
    credentials:
      key: <your-access-key>
```

### storage.azure.account

*Type*: string <br>
*Required*: YES

The name of your storage account.

### storage.azure.container

*Type*: string <br>
*Required*: YES

The name of the storage container. See the  [Container names](https://docs.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) for naming conventions.

### storage.azure.endpointUrl

*Type*: string <br>
*Required*: NO

The URL to access the data in Microsoft Azure Blob Storage. The default value is `https://<storage-account>.blob.core.windows.net`.

### storage.azure.prefix

*Type*: string <br>
*Required*: NO

The path (sub-folder) to the backups inside the container. If undefined, backups are stored in the container root directory.

### storage.azure.credentials.key

*Type*: string <br>
*Required*: YES

Your access key to authorize access to data in your storage account.


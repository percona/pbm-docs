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
    endpointUrlMap: 
      "node01:2017": <string>
      "node02:2017": <string>
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
      session-token: <string>
    uploadPartSize: <int>
    maxUploadParts: <int>
    storageClass: STANDARD
    serverSideEncryption:
      sseAlgorithm: aws:kms
      kmsKeyID: <your-kms-key-here>
      sseCustomerAlgorithm: AES256
      sseCustomerKey: <your_encryption_key>
    retryer:
      numMaxRetries: 3
      minRetryDelay: 30ms
      maxRetryDelay: 5m
```

### storage.s3.provider


*Type*: string <br>
*Required*: NO

The storage provider’s name. 

Supported values: `aws`

### storage.s3.bucket


*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [AWS Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) for bucket name requirements.

### storage.s3.region

*Type*: string <br>
*Required*: YES (for AWS)

The location of the storage bucket.
Use the [AWS region list](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) and [GCS region list](https://cloud.google.com/storage/docs/locations) to define the bucket region

### storage.s3.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### storage.s3.endpointUrl

*Type*: string <br>
*Required*: YES (for MinIO)

The URL to access the bucket. 

### storage.s3.endpointUrlMap

*Type*: array of strings <br>
*Required*: NO

The list of custom paths for `pbm-agents` on different servers to the same storage. Use this option if `pbm-agents` reside on servers hidden behind different network configurations. Read more in the [Support for multiple endpoints to the same S3 storage](../details/s3-storage.md#support-for-multiple-endpoints-to-the-same-s3-storage) section. Supported for Amazon S3 and Microsoft Azure Blob storages. Available with version 2.8.0.

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
*Example*: STANDARD

The [storage class :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/) assigned to objects stored in the S3 bucket. If not provided, the `STANDARD` storage class will be used. This option is available in Percona Backup for MongoDB as of v1.7.0.

### storage.s3.debugLogLevels

*Type*: string <br>
*Required*: NO

Enables S3 debug logging for different types of S3 requests. S3 log messages are printed in the `pbm logs` output.

Supported values are: `LogDebug`, `Signing`, `HTTPBody`, `RequestRetries`, `RequestErrors`, `EventStreamBody`.

To specify several event types, separate them by comma. To lean more about the event types, see [the documentation :octicons-link-external-16:](https://pkg.go.dev/github.com/aws/aws-sdk-go@v1.40.7/aws#LogLevelType)

When undefined, no S3 debug logging is performed.

### storage.s3.insecureSkipTLSVerify

*Type*: bool <br>
*Required*: NO <br>
*Default*: False

Disables the TLS verification of the S3 storage. This allows Percona Backup for MongoDB to upload data to S3-like storages that use self-issued TLS certificates. Available in Percona Backup for MongoDB as of version 1.7.0.

!!! warning 
    
    Use this option with caution as it might leave a hole for man-in-the-middle attacks.

## GCS type storage options

### gcs.bucket

*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [GCS bucket naming guidelines](https://cloud.google.com/storage/docs/naming-buckets#requirements) for bucket name requirements.

### gcs.chunkSize

*Type*: string <br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket in a single request. Larger data chunks will be split over multiple requests. Default data chunk size is 16MB.

### gcs.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### gcs.endpointUrl

*Type*: string <br>
*Required*: NO

The URL to access the bucket. The default value is `https://storage.googleapis.com`. 

### gcs.credentials.projectId

*Type*: string <br>
*Required*: YES

Project resource ID - a unique identifier for the project resource. Read more about projects in Google Cloud storage in the [Resource hierarchy :octicons-link-external-16:](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#projects) documentation.

### gcs.credentials.privateKey

*Type*: string <br>
*Required*: YES

The private key of the service account used to authenticate the request.

### gcs.credentials.HMACAccessKey

*Type*: string <br>
*Required*: YES

The HMAC access key associated with your service account. The access key is used to authenticate the request to GCS via the XML API. 

### gcs.credentials.HMACSecretKey

*Type*: string <br>
*Required*: YES

A 40-character Base-64 encoded string that is linked to a specific HMAC access ID. You receive the secret when you create an HMAC key. It is used to create signatures as part of the authentication process. 

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

The key management mode for [server-side encryption with customer-provided keys (SSE-C)](../details/s3-storage.md#server-side-encryption).

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
*Default*: 30ms

The minimum time to wait before the next retry, specified as a *time.Duration*. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided. Available in Percona Backup for MongoDB as of 1.7.0.

### retryer.maxRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 5m

The maximum time to wait before the next retry, specified as a *time.Duration*. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided. Available in Percona Backup for MongoDB as of 1.7.0.

## GCS type storage options

```yaml
storage:
 type: gcs
 gcs:
    bucket: pbm-testing
    chunkSize: 16777216
    prefix: pbm/test
    credentials:
      clientEmail: <your-client-email-here>
      privateKey: <your-private-key-here>
      HMACAccessKey: <your-HMAC-key-here>
      HMACSecret: <your-HMAC-secret-here>
```

### gcs.bucket

*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [GCS bucket naming guidelines](https://cloud.google.com/storage/docs/naming-buckets#requirements) for bucket name requirements.

### gcs.chunkSize

*Type*: string <br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket in a single request. Larger data chunks will be split over multiple requests. Default data chunk size is 16MB.

### gcs.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### gcs.endpointUrl

*Type*: string <br>
*Required*: NO

The URL to access the bucket. The default value is `https://storage.googleapis.com`. 

### gcs.credentials.clientEmail

*Type*: string <br>
*Required*: YES

The clientEmail represents the email address associated with the service account. It is used to identify the service account when making  API requests to Google Cloud services. These credentials are used to authenticate to Cloud Storage via JSON API.

### gcs.credentials.privateKey

*Type*: string <br>
*Required*: YES

The private key of the service account used to authenticate the request. These credentials are used to authenticate to Cloud Storage via JSON API.

### gcs.credentials.HMACAccessKey

*Type*: string <br>
*Required*: YES

An access key associated with a service account and used to create signatures using the HMAC-SHA256 signing algorithm. These signatures are then included in requests to the Cloud Storage XML API.

### gcs.credentials.HMACSecret

*Type*: string <br>
*Required*: YES

A 40-character Base-64 encoded string that is linked to a specific HMAC access key. This secret is used to create signatures to authenticate users in the Cloud Storage via XML API.

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

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### storage.azure.credentials.key

*Type*: string <br>
*Required*: YES

Your access key to authorize access to data in your storage account.


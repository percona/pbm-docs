# Remote backup storage options

## Common options

### storage.type

*Type*: string <br>
*Required*:     YES   

Remote backup storage type. Supported values: `s3`, `minio`, `gcs`, `filesystem`, `azure`.

## AWS S3 storage options

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
    maxObjSizeGB: 5018
```

### storage.s3.provider


*Type*: string <br>
*Required*: NO

The storage provider's name. This field is deprecated.


### storage.s3.bucket


*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [AWS Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) for bucket name requirements.

### storage.s3.region

*Type*: string <br>
*Required*: YES (for AWS)

The location of the storage bucket.
Use the [AWS region list](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)  to define the bucket region

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

The list of custom paths for `pbm-agents` on different servers to the same storage. Use this option if `pbm-agents` reside on servers hidden behind different network configurations. Read more in the [Support for multiple endpoints to the same S3 storage](../details/endpoint-map.md) section. Supported for Amazon S3 and Microsoft Azure Blob storages. Available with version 2.8.0.

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

The [storage class :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/) assigned to objects stored in the S3 bucket. If not provided, the `STANDARD` storage class will be used. 

### storage.s3.debugLogLevels

*Type*: string <br>
*Required*: NO

Enables S3 debug logging for different types of S3 requests. S3 log messages are printed in the `pbm logs` output.

Starting with version 2.10.0, PBM uses AWS SDK v2. The AWS SDK v1 values are deprecated. They are kept for backward compatibility.

Please find the mapping table below:

| AWS SDK v1 value | AWS SDK v2 value |
|------------------|------------------|
| `LogDebug`       | `Request` <br> `Response`|
| `Signing`        | `Signing`|
| `HTTPBody`       | `RequestWithBody` <br> `ResponseWithBody`|
| `RequestRetries` | `DebugWithRequestRetries`|
| `RequestErrors`  | `DebugWithRequestErrors`|
| `EventStreamBody`| `RequestWithBody` <br> `ResponseWithBody`|


To specify several event types, separate them by comma. To lean more about the event types, see [the documentation :octicons-link-external-16:](https://pkg.go.dev/github.com/aws/aws-sdk-go@v1.40.7/aws#LogLevelType)

When undefined, no S3 debug logging is performed.

### storage.s3.insecureSkipTLSVerify

*Type*: bool <br>
*Required*: NO <br>
*Default*: False

Disables the TLS verification of the S3 storage. This allows Percona Backup for MongoDB to upload data to S3-like storages that use self-issued TLS certificates.

!!! warning 
    
    Use this option with caution as it might leave a hole for man-in-the-middle attacks.

## Server-side encryption options

### storage.s3.serverSideEncryption.sseAlgorithm

*Type*: string <br>
*Required*: NO 

The key management mode used for server-side encryption with the encryption keys stored in AWS KMS.

Supported value: `aws:kms`

### storage.s3.serverSideEncryption.kmsKeyID

*Type*: string <br>
*Required*: NO

Your customer-managed key stored in the AWS KMS.

### storage.s3.serverSideEncryption.sseCustomerAlgorithm

*Type*: string <br>
*Required*: NO 

The key management mode for [server-side encryption with customer-provided keys (SSE-C)](../details/s3-storage.md#server-side-encryption).

Supported value: `AES256`

### storage.s3.serverSideEncryption.sseCustomerKey

*Type*: string <br>
*Required*: NO

Your custom encryption key. This key is not stored on the S3 storage side. Thus, it is your responsibility to track what data is encrypted with what key and for storing the key. 

## Upload retry options

### storage.s3.retryer.numMaxRetries

*Type*: int <br>
*Required*: NO <br>
*Default*: 3

The maximum number of retries to upload data to S3 storage. A zero value means no retries will be performed. 

### storage.s3.retryer.minRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 30ms

The minimum time to wait before the next retry, specified as a *time.Duration*. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided. 

### storage.s3.retryer.maxRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 5m

The maximum time to wait before the next retry, specified as a *time.Duration*. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided. 

### storage.s3.maxObjSizeGB

*Type*: float64 <br>
*Required*: NO <br>
*Default*: 5018

The maximum file size to be stored on the backup storage. If the file to upload exceeds this limit, PBM splits it in pieces, each of which falls within the limit. Read more about [Managing large backup files](../features/split-merge-backup.md).

## MinIO type storage options

You can use this storage type for other S3-compatible storage services

```yaml
storage:
  type: minio
  minio:
    region: <string>
    bucket: <string>
    prefix: <string>
    endpoint: <string>
    endpointMap: 
      "node01:2017": <string>
      "node02:2017": <string>
    secure: false
    insecureSkipTLSVerify: false
    forcePathStyle: false
    credentials:
      access-key-id: <string>
      secret-access-key: <string>
      session-token: <string>
      signature-ver: V4
    partSize: 10485760 (10 MB)
    retryer:
      numMaxRetries: 10
    maxObjSizeGB: 5018
    debugTrace: false 
```

### storage.minio.region

*Type*: string <br>
*Required*: NO

The location of the storage bucket. Use the [AWS region list](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) to define the bucket region. If not specified, the default `us-east-1` region is used.

### storage.minio.bucket


*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [AWS Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) for bucket name requirements.

### storage.minio.prefix


*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket’s root directory.

### storage.minio.endpoint

*Type*: string <br>
*Required*: YES

The network address (URL or IP:port) where your MinIO server is accessible.

### storage.minio.endpointMap

*Type*: array of strings <br>
*Required*: NO

A mapping of custom endpoints for `pbm-agents` on different servers to the same MinIO storage. Use this option if `pbm-agents` reside on servers hidden behind different network configurations. Read more in the [Support for multiple endpoints to the same S3 storage](../details/endpoint-map.md) section. Supported for Amazon S3, MinIO, and Microsoft Azure Blob storages. Available with version 2.8.0.

### storage.minio.secure

*Type*: boolean <br>
*Required*: NO <br>
*Default*: false

Defines whether to use HTTP or HTTPS protocol for communication between PBM and S3 storage. Default: `false`.

### storage.minio.insecureSkipTLSVerify

*Type*: boolean <br>
*Required*: NO <br>
*Default*: false

Disables the TLS verification of the MinIO / S3-compatible storage. This allows Percona Backup for MongoDB to upload data to MinIO / S3-compatible storages that use self-issued TLS certificates. Use it with caution as it might leave a hole for man-in-the-middle attacks.

### storage.minio.forcePathStyle

*Type*: boolean <br>
*Required*: NO <br>
*Default*: false

Enforces the use of [path style access](../reference/glossary.md#path-style-access-to-the-storage) to the storage. Default is `false` which means PBM uses the [virtual-hosted-style](../reference/glossary.md#virtual-hosted-style-access) access to the storage

### storage.minio.credentials.access-key-id

*Type*: string<br>
*Required*: YES

Your access key to the storage bucket.

### storage.minio.credentials.secret-access-key

*Type*: string<br>
*Required*: YES

The key to sign your programmatic requests to the storage bucket.

### storage.minio.credentials.session-token

*Type*: string<br>
*Required*: NO

The MinIO session token used to validate the temporary security credentials for accessing the storage.

### storage.minio.credentials.signature-ver

*Type*: string<br>
*Required*: NO<br>
*Default*: V4

Specifies the AWS Signature version to use for authentication. Accepted values: `V2`, `V4`. 

Allows using the deprecated AWS Signature version 2 for backward compatibility with storages that don't support Signature version 4. Default: `V4`.

### storage.minio.partSize

*Type*: int<br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket. Default: 10MB.

### storage.minio.retryer.numMaxRetries

*Type*: int<br>
*Required*: NO<br>
*Default*: 10

The maximum number of retries to upload data to MinIO / S3-compatible storage. A zero value means no retries will be performed.

### storage.minio.maxObjSizeGB

*Type*: float64<br>
*Required*: NO<br>
*Default*: 5018

The maximum file size to be stored on the backup storage. If the file to upload exceeds this limit, PBM splits it in pieces, each of which falls within the limit. Read more about [Managing large backup files](../features/split-merge-backup.md).

### storage.minio.debugTrace

*Type*: boolean<br>
*Required*: NO

If set to `true`, outputs all http communication trace in PBM log. Default: false.

## GCS type storage options

```yaml
storage:
 type: gcs
 gcs:
    bucket: pbm-testing
    chunkSize: <int>
    prefix: pbm/test
    credentials:
      clientEmail: <your-client-email-here>
      privateKey: <your-private-key-here>
      hmacAccessKey: <your-HMAC-key-here>
      hmacSecret: <your-HMAC-secret-here>
    maxObjSizeGB: 5018
```

### storage.gcs.bucket

*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [GCS bucket naming guidelines](https://cloud.google.com/storage/docs/naming-buckets#requirements) for bucket name requirements.

### storage.gcs.chunkSize

*Type*: string <br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket in a single request. Larger data chunks will be split over multiple requests. Default data chunk size is 10MB.

### storage.gcs.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### storage.gcs.credentials.clientEmail

*Type*: string <br>
*Required*: YES

The email address that uniquely identifies your service account in GCS.

### storage.gcs.credentials.privateKey

*Type*: string <br>
*Required*: YES

The private key of the service account used to authenticate the request.

### storage.gcs.credentials.hmacAccessKey

*Type*: string <br>
*Required*: YES

The HMAC access key associated with your service account. The access key is used to authenticate the request to GCS via the XML API. 

The use of HMAC keys is deprecated starting with version 2.12.0. Use the `storage.gcs.credentials.clientEmail` and `storage.gcs.credentials.privateKey` instead.

### storage.gcs.credentials.hmacSecret

*Type*: string <br>
*Required*: YES

A 40-character Base-64 encoded string that is linked to a specific HMAC access ID. You receive the secret when you create an HMAC key. It is used to create signatures as part of the authentication process. 

The use of HMAC keys is deprecated starting with version 2.12.0. Use the `storage.gcs.credentials.clientEmail` and `storage.gcs.credentials.privateKey` instead.

### storage.gcs.retryer.backoffInitial

*Type*: time.Duration <br>
*Required*: NO
*Default*: 1s

The time to wait to make an initial retry, specified as a time.Duration. Units like ms, s, etc., are supported. Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

Defaults to nanoseconds if no unit is provided.

### storage.gcs.retryer.backoffMax

*Type*: time.Duration <br>
*Required*: NO
*Default*: 30s

The maximum amount of time between retries, in seconds. Units like ms, s, etc., are supported. Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

Defaults to nanoseconds if no unit is provided.

### storage.gcs.retryer.backoffMultiplier

*Type*: int <br>
*Required*: NO
*Default*: 2

Each time PBM fails and tries again, it increases the wait time by multiplying it by this number. Default value is 2 sec.

For example, if the first wait time is 1 second, the next will be 2 seconds, then 4 seconds, and so on, until it reaches the maximum. Default value is 2 sec.

### storage.gcs.retryer.maxAttempts

*Type*: int <br>
*Required*: NO
*Default*: 5

The maximum number of retries to upload data to GCS storage. A zero value means no retries will be performed. Available starting with version 2.12.0.

### storage.gcs.retryer.chunkRetryDeadline

*Type*: time.Duration <br>
*Required*: NO
*Default*: 32s

When you upload large files to GCS using resumable uploads, the data is sent in chunks. If a chunk fails to upload due to a network issue, timeout or transient error, GCS will retry sending that chunk.

The `chunkRetryDeadline` sets a time limit in seconds for how long GCS will keep retrying a failed chunk. Once this deadline is reached, GCS stops retrying and marks the upload as failed.

Units like ms, s, etc., are supported. Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

Defaults to nanoseconds if no unit is provided.

Available starting with version 2.12.0.

### storage.gcs.maxObjSizeGB

*Type*: float64 <br>
*Required*: NO <br>
*Default*: 5018

The maximum file size to be stored on the backup storage. If the file to upload exceeds this limit, PBM splits it in pieces, each of which falls within the defined limit. Read more about [Managing large backup files](../features/split-merge-backup.md).

### storage.gcs.debugTrace

*Type*: boolean
*Required*: NO

When set to `true`, activates detailed logging of HTTP requests and responses between PBM and GCS. 

## Filesystem storage options

```yaml
storage:
  type: filesystem
  filesystem:
    path: <string>
  maxObjSizeGB: 5018
```

### storage.filesystem.path

*Type*: string <br>
*Required*: YES

The path to the backup directory

### storage.filesystem.maxObjSizeGB

*Type*: float64 <br>
*Required*: NO <br>
*Default*: 5018

The maximum file size to be stored on the backup storage. If the file to upload exceeds this limit, PBM splits it in pieces, each of which falls within the defined limit. Read more about [Managing large backup files](../features/split-merge-backup.md).

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
    maxObjSizeGB: 194560
    retryer:
      numMaxRetries: 3
      minRetryDelay: 800ms
      maxRetryDelay: 60s
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

### storage.azure.endpointUrlMap

*Type*: object (host:port -> endpoint URL) <br>
*Required*: NO

A mapping of custom endpoint URLs for `pbm-agents` on different servers to the same remote storage. Use this option if `pbm-agents` reside on servers hidden behind different network configurations. Read more in the [Support for multiple endpoints to the same remote storage](../details/endpoint-map.md) section. Available with version 2.8.0.


### storage.azure.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### storage.azure.credentials.key

*Type*: string <br>
*Required*: YES

Your access key to authorize access to data in your storage account.

### storage.azure.maxObjSizeGB

*Type*: float64 <br>
*Required*: NO <br>
*Default*: 194560

The maximum file size to be stored on the backup storage. If the file to upload exceeds this limit, PBM splits it in pieces, each of which falls within the defined limit. Read more about [Managing large backup files](../features/split-merge-backup.md).

### storage.azure.retryer.numMaxRetries

*Type*: int <br>
*Required*: NO <br>
*Default*: 3

The maximum number of retries to upload data to Microsoft Azure storage. A zero value means no retries will be performed.

### storage.azure.retryer.minRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 800ms

The minimum time to wait before the next retry, specified as a `time.Duration`. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided.


### storage.azure.retryer.maxRetryDelay

*Type*: time.Duration <br>
*Required*: NO <br>
*Default*: 60s

The maximum time to wait before the next retry, specified as a `time.Duration`. Units like ms, s, etc., are supported. Defaults to nanoseconds if no unit is provided.

## Alibaba Cloud OSS storage options

```yaml
storage:
  type: oss
  oss:
    region: <string>
    endpointUrl: <string>
    bucket: <string>
    prefix: <string>
    credentials:
      accessKeyId: 'LTAI5t...EXAMPLE'
      accessKeySecret: 'n4VpW...EXAMPLE'
      securityToken: 'CAIS...EXAMPLE'
      roleArn: acs:ram::1234567890123456:role/db-backup-role
      sessionName: <string>
    serverSideEncryption:
      sseAlgorithm: <string>
      KMSMasterKeyID: <string>
      KMSDataEncryption: <string>
    uploadPartSize: <int>
    maxUploadParts: <int>
    connectTimeout: 5s
    maxObjSizeGB: 48700
    retryer:
      maxAttempts: 5
      maxBackoff: 300s
      baseDelay: 30ms
```

### storage.oss.region

*Type*: string <br>
*Required*: YES

The region where your OSS bucket is located. Refer to the [OSS regions and endpoints](https://www.alibabacloud.com/help/en/oss/user-guide/regions-and-endpoints) for the list of available regions.

### storage.oss.endpointUrl

*Type*: string <br>
*Required*: YES

A domain name to access OSS over the Internet. The endpoint must correspond to the region that you selected when you created the bucket. 

### storage.oss.bucket

*Type*: string <br>
*Required*: YES

The name of the storage bucket. See the [OSS bucket naming rules](https://www.alibabacloud.com/help/en/oss/user-guide/bucket-naming-conventions) for bucket name requirements.

### storage.oss.prefix

*Type*: string <br>
*Required*: NO

The path to the data directory in the bucket. If undefined, backups are stored in the bucket's root directory.

### storage.oss.credentials.accessKeyId

*Type*: string <br>
*Required*: YES

The Access Key ID associated with the RAM user used to access Alibaba Cloud OSS.

### storage.oss.credentials.accessKeySecret

*Type*: string <br>
*Required*: YES

The Access Key Secret associated with the RAM user used to access Alibaba Cloud OSS. The secret is used to encrypt and verify the signature string.

### storage.oss.credentials.securityToken

*Type*: string <br>
*Required*: YES when using temporary credentials

The security token that is used together with temporary access key ID and access key secret. You receive the token when you request temporary access credentials by using the [Security Token Service](https://www.alibabacloud.com/help/en/ram/product-overview/what-is-sts).

### storage.oss.credentials.roleArn

*Type*: string <br>
*Required*: NO

The Alibaba Cloud Resource Name (ARN) of the RAM role to assume. PBM uses this role to obtain required permissions for accessing OSS resources.

### storage.oss.credentials.sessionName

*Type*: string <br>
*Required*: NO

Identifier for the assumed role session

### storage.oss.serverSideEncryption.sseAlgorithm

*Type*: string <br>
*Required*: NO 

The encryption algorithm used to encrypt data before storing it in OSS. Supported values: `AES256`, `KMS`, `SM4`. Default: `AES256`

### storage.oss.serverSideEncryption.kmsMasterKeyId 

*Type*: string <br>
*Required*: YES (when using a custom KMS key)

The ID of the customer master key used for encryption.

### storage.oss.serverSideEncryption.kmsDataEncryption

*Type*: string <br>
*Required*: NO

The encryption algorithm for encrypting data when SSE-KMS is used. Can be set only when `storage.oss.serverSideEncryption.sseAlgorithm` is set to `KMS`.

Supported values: `AES256`, `SM4`. Default: `AES256`

### storage.oss.uploadPartSize

*Type*: int <br>
*Required*: NO

The size of data chunks in bytes to be uploaded to the storage bucket. Default: 10MB

Percona Backup for MongoDB automatically increases the `uploadPartSize` value if the size of the file to be uploaded exceeds the max allowed file size. (The max allowed file size is calculated with the default values of `uploadPartSize` \* [`maxUploadParts`](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#pkg-constants) and is appr. 97,6 GB).

The `uploadPartSize` value is printed in the `pbm-agent` log.

By setting this option, you can manually adjust the size of data chunks if Percona Backup for MongoDB failed to do it for some reason. The defined `uploadPartSize` value overrides the default value and is used for calculating the max allowed file size

### storage.oss.maxUploadParts

*Type*: int <br>
*Required*: NO <br>
*Default*: 10,000

The maximum number of data chunks to be uploaded to the storage bucket. Default: 10,000

By setting this option, you can override the value defined in the [Multipart upload](https://www.alibabacloud.com/help/en/oss/developer-reference/multipart-upload-3?spm=a3c0i.29367734.6737026690.4.43067d3faLVHMa) method.

It can be useful to specify a smaller number of chunks for multipart uploads.

The `maxUploadParts` value is printed in the pbm-agent log.


### storage.oss.connectTimeout

*Type*: int <br>
*Required*: NO <br>
*Default*: 5s

The connection timeout in seconds when PBM connects to the OSS storage. Default value is 5 seconds.

### storage.oss.retryer.maxAttempts

*Type*: int <br>
*Required*: NO <br>
*Default*: 300

The maximum number of retry attempts for failed requests to the OSS storage. Default value is 5.

### storage.oss.retryer.maxBackoff

*Type*: int <br>
*Required*: NO <br>
*Default*: 300s

The maximum time in seconds to wait between retry attempts for failed requests to the OSS storage. Default value is 5 minutes (300 seconds).

### storage.oss.retryer.baseDelay

*Type*: int <br>
*Required*: NO <br>
*Default*: 30ms

The initial delay before the first retry attempt. Default value is 30 ms.

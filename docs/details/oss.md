---
title: Alicloud OSS storage
---

Percona Backup for MongoDB supports using [Alicloud OSS](https://www.alibabacloud.com/product/object-storage-service) for storing backups.

To use Alicloud OSS, specify the following in the PBM configuration file:

```yaml
storage:
  type: oss
  oss:
    region: <region>
    endpointUrl: <endpointUrl>
    bucket: <bucket_name>
    prefix: <prefix>
    credentials:
      accessKeyId: <access_key_id>
      accessKeySecret: <access_key_secret>
```

## Configuration options

| Name | Description | Default |
| :--- | :--- | :--- |
| `region` | The region where your bucket is located. | `ap-southeast-5` |
| `endpointUrl` | The endpoint URL for Alicloud OSS. | |
| `bucket` | The name of your bucket. | |
| `prefix` | The prefix for your backups. | |
| `credentials.accessKeyId` | The access key ID for your Alicloud account. | |
| `credentials.accessKeySecret` | The access key secret for your Alicloud account. | |
| `credentials.securityToken` | The security token for your Alicloud account. | |
| `credentials.roleArn` | The ARN of the RAM role to assume. | |
| `credentials.sessionName` | The session name for the assumed role. | |
| `uploadPartSize` | The size of each part for multipart uploads. | `10485760` (10MB) |
| `maxUploadParts` | The maximum number of parts for multipart uploads. | `10000` |
| `retryer.maxAttempts` | The maximum number of retry attempts for failed requests. | `5` |
| `retryer.maxBackoff` | The maximum backoff time for retries. | `300s` |
| `retryer.baseDelay` | The base delay for retries. | `30ms` |
| `connectTimeout` | The connection timeout for requests. | `5s` |

You can get the `accessKeyId` and `accessKeySecret` from the Alicloud console.

For more information on how to configure storage, see [PBM remote storage configuration](storage-configuration.md).

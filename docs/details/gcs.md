# Google Cloud Storage (GCS)

You can use Google Cloud Storage (GCS) as a remote backup storage for Percona Backup for MongoDB. 

!!! admonition ""

    Starting from version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. See how to [adjust your PBM configuration to use GCS](#adjust-pbm-configuration-to-use-gcs) after the upgrade.


PBM supports communication with GCS via the JSON API and XML API. The preferred approach is to use the JSON API with a service account. HMAC keys are mainly useful for compatibility with S3-style APIs.

!!! warning "HMAC keys support deprecation"

    Starting with version 2.12.0, HMAC keys support is deprecated. We encourage you to use GCS connection type with native JSON keys.

To use GCS, you need the following:

* [create a service account :octicons-link-external-16:](https://cloud.google.com/iam/docs/service-accounts-create#iam-service-accounts-create-console) 
* add keys for the service account:

    * [add JSON keys :octicons-link-external-16:](https://cloud.google.com/iam/docs/keys-create-delete#creating) or
    * [add HMAC keys :octicons-link-external-16:](https://cloud.google.com/storage/docs/authentication/managing-hmackeys). This method is deprecated and not recommended for use

* [create a bucket](#create-a-bucket)
* [add the GCS configuration to PBM](#configuration-example) 

## Create a bucket

1. Install and configure the [gcloud CLI :octicons-link-external-16:](https://cloud.google.com/sdk/docs/install)

2. Create a bucket

    ```bash
    gcloud storage buckets create my-gcs-bucket --location=US
    ```
      
3. Verify the bucket creation

    ```bash
    gcloud storage buckets list
    ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

## Configuration example

You can find [the configuration file template :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

=== "using JSON keys"

    ```yaml
    storage:
     type: gcs
     gcs:
         bucket: pbm-testing
         prefix: pbm/test
         credentials:
           clientEmail: <your-service-account-email-here>
           privateKey: <your-private-key-here>
    ```

=== "using HMAC keys (deprecated)"

	```yaml
	storage:
	 type: gcs
	 gcs:
		 bucket: pbm-testing
		 prefix: pbm/test
		 credentials:
		   hmacAccessKey: <your-access-key-id-here>
		   hmacSecret: <your-secret-key-here>
	```


## Enable parallel uploads

Parallel uploads can reduce the time required to send large backup files to Google Cloud Storage. The improvement depends on the available network bandwidth, storage performance, and resources on the host running pbm-agent.

To enable parallel uploads, use the GCS gRPC client and set the number of concurrent uploads:

Parallel uploads are available only with the gRPC client. To enable them, set:

* `clientType` to `grpc`
* `parallelUploadConcurrency` to a value greater than `1`

The following example enables four concurrent uploads:

```yaml
storage:
  type: gcs
  gcs:
    bucket: <bucket-name>
    prefix: <optional-prefix>
    clientType: grpc
    parallelUploadConcurrency: 4
    chunkSize: 16MB
    credentials:
      workloadIdentity: true
       #storage:
  type: gcs
  gcs:
    bucket: <bucket-name>
    prefix: <optional-prefix>
    clientType: grpc
    parallelUploadConcurrency: 4
    chunkSize: 16MB
    credentials:
      # Use your existing service account or Workload Identity configuration.
```

You can use parallel uploads with either Workload Identity or service account credentials. Keep the credentials section that matches your authentication method.

Apply the configuration:

```sh
pbm config --file pbm_config.yaml
```
### Configure upload concurrency

The `parallelUploadConcurrency` option controls how many parts PBM uploads at the same time.

A higher value can improve throughput when network bandwidth and storage performance are available. It also increases the number of concurrent requests and the resources used by the upload. Start with a moderate value, such as `4`, and measure backup performance before increasing it.

If p`arallelUploadConcurrency` is omitted or set to `1`, PBM uses a standard upload.

### Configure the part size

For parallel uploads, `chunkSize` defines the size of each temporary part. If you do not set it, PBM uses a default part size of 16 MiB.

Larger parts reduce the number of temporary objects and compose operations. Smaller parts give PBM more work to distribute across concurrent upload operations. Choose a value that fits your backup size, available memory, and network capacity.

!!! note
    Parallel uploads require both `clientType: grpc` and a `parallelUploadConcurrency` value greater than `1`. The JSON client does not support this feature. If you configure parallel uploads with the JSON client, PBM performs a standard upload instead.

!!! warning
    - Parallel upload support in the upstream Google Cloud Storage Go client is experimental. Test the configuration with representative backup sizes before using it in production.
    - Parallel uploads create temporary objects in the destination bucket. The credentials used by PBM must have permission to delete these objects. An interrupted upload can leave temporary objects behind. Consider configuring an [Object Lifecycle Management rule :octicons-link-external-16:](https://docs.cloud.google.com/storage/docs/lifecycle){:target="_blank"} to remove abandoned temporary objects.
    - Review your bucket settings before enabling this feature. Retention policies, default object holds, soft delete, and Object Versioning can prevent immediate cleanup or increase storage costs. See [Parallel composite uploads :octicons-link-external-16:](https://docs.cloud.google.com/storage/docs/parallel-composite-uploads){:target="_blank"} for details.

For the upstream client configuration and defaults, see ParallelUploadConfig in the Google Cloud Storage Go client :octicons-link-external-16:{="_blank"}.

## Adjust PBM configuration to use GCS

Starting with version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. If you are upgrading from an earlier version, you need to adjust your PBM configuration as follows:

1. Change the `storage.type` from `s3` to `gcs`.
2. Change the `storage.s3` section to `storage.gcs` and adjust the parameters accordingly. See the [Configuration example](#configuration-example) above. Select the option depending on the authentication method you use.

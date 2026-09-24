# Google Cloud Storage (GCS)

You can use Google Cloud Storage (GCS) as a remote backup storage for Percona Backup for MongoDB. 

!!! admonition ""

    Starting from version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. See how to [adjust your PBM configuration to use GCS](#adjust-pbm-configuration-to-use-gcs) after the upgrade.

PBM communicates with GCS through the JSON API by default. You can configure the gRPC client for both standard and parallel uploads. PBM authenticates using either a service account or Workload Identity.

!!! warning "HMAC authentication removed in PBM 2.16.0"
    PBM 2.16.0 removes support for authenticating to GCS with HMAC keys. Before upgrading, replace `hmacAccessKey` and `hmacSecret` in your PBM configuration with `clientEmail` and `privateKey`. PBM 2.16.0 treats the HMAC options as unrecognized configuration fields and cannot access the backup storage until you update the configuration.

To use GCS, you need the following:

* [create a service account :octicons-link-external-16:](https://cloud.google.com/iam/docs/service-accounts-create#iam-service-accounts-create-console) 
* For static service account authentication, [add JSON keys :octicons-link-external-16:](https://cloud.google.com/iam/docs/keys-create-delete#creating). Alternatively, configure [Workload Identity authentication](workload-identity-auth.md).
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

## Parallel uploads to GCS

Parallel uploads can reduce the time required to send large backup files to Google Cloud Storage. The improvement depends on the available network bandwidth, storage performance, and resources on the host running `pbm-agent`.

This feature applies only when PBM writes data to GCS. It does not change how PBM downloads backup data during a restore.

!!! warning "Experimental feature"
    - Parallel upload support in the Google Cloud Storage Go client is experimental. The upstream API may change in future releases and is not yet recommended for production use.
    - The current SDK implementation can leave temporary part objects under the `gcs-go-sdk-pu-tmp` prefix. Configure an external cleanup job to remove these objects. Schedule the cleanup outside your backup window so that it does not remove parts from an active upload.
    - Evaluate parallel uploads with a representative backup before enabling them in production. For details, see the upstream [`ParallelUploadConfig` documentation :octicons-link-external-16:](https://pkg.go.dev/cloud.google.com/go/storage#ParallelUploadConfig){:target="_blank"}.

### How parallel uploads work

The Google Cloud Storage client divides a large backup object into parts and uploads multiple parts concurrently. The `chunkSize` option controls the size of each part. The `parallelUploadConcurrency` option controls the maximum number of concurrent uploads.

GCS composes the uploaded parts into the final backup object. The client then makes a best-effort attempt to remove the temporary part objects. If the upload process exits unexpectedly, some temporary objects may remain in the bucket.

For details, see the upstream [Parallel Uploads documentation :octicons-link-external-16:](https://pkg.go.dev/cloud.google.com/go/storage#hdr-Parallel_Uploads){:target="_blank"}.

### Before you start

Make sure that:

* You are running PBM 2.16.0 or later.
* Your PBM configuration uses the native GCS storage type.
* The credentials used by PBM can create, compose, and delete objects in the GCS bucket.
* The destination bucket does not have settings that prevent PBM from deleting temporary objects.

Parallel uploads work with both service account credentials and Workload Identity authentication. Keep the authentication configuration already defined for your GCS storage. For Workload Identity configuration, see [Workload Identity authentication](workload-identity-auth.md).


### Configure parallel uploads

Set `clientType` to `grpc` and `parallelUploadConcurrency` to a value greater than `1`:


```yaml
storage:
  type: gcs
  gcs:
    bucket: <bucket-name>
    prefix: <optional-prefix>
    clientType: grpc
    parallelUploadConcurrency: 4
```
In this example, `chunkSize` is omitted, so PBM uses the 16 MiB default for parallel uploads. PBM uploads up to four parts concurrently.

Keep your existing `credentials` section in the configuration file.
{.power-number}

1. Apply the configuration:

    ```bash
    pbm config --file pbm_config.yaml
    ```

2. Check the active configuration:

    ```bash
    pbm config --list
    ```

### Configuration options

| **Option** | **Description**| **Default**|
|------------|----------------|-------------|
| `clientType`               | GCS client used by PBM. Parallel uploads require `grpc`. If you use `json`, PBM performs a standard upload. | `json`                       |
| `parallelUploadConcurrency`| Maximum number of parts PBM uploads concurrently. A value greater than 1 enables parallel uploads when `clientType` is `grpc`. | 0 (parallel uploads disabled)|
| `chunkSize`                | Size, in bytes, of each data chunk sent to GCS. If you omit this option, PBM selects the default based on the upload mode.| 10485760 bytes (10 MiB) for standard uploads; 16777216 bytes (16 MiB) for parallel uploads|

For the complete list of GCS settings, see [Remote backup storage options](../reference/configuration-options.md).

### Choose a concurrency value

A higher concurrency value does not always produce a faster backup. The value that works best depends on the available resources and workload.

### Disable parallel uploads

To disable parallel uploads, remove `parallelUploadConcurrency` from the configuration or set it to a value that is not greater than `1`.

You can continue using the `grpc` client with parallel uploads disabled. You do not need to switch back to the `json` client.

## Adjust PBM configuration to use GCS

Starting with version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. If you are upgrading from an earlier version, you need to adjust your PBM configuration as follows:

1. Change the `storage.type` from `s3` to `gcs`.
2. Change the `storage.s3` section to `storage.gcs` and adjust the parameters accordingly. See the [Configuration example](#configuration-example) above.

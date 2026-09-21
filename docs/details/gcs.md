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

## Parallel uploads to GCS

Parallel uploads can reduce the time required to send large backup files to Google Cloud Storage. The improvement depends on the available network bandwidth, storage performance, and resources on the host running `pbm-agent`.

This feature applies only when PBM writes data to GCS. It does not change how PBM downloads backup data during a restore.

### Before you start

Make sure that:

* You are running PBM 2.16.0 or later.
* Your PBM configuration uses the native GCS storage type.
* The credentials used by PBM can create, compose, and delete objects in the GCS bucket.
* The destination bucket does not have settings that prevent PBM from deleting temporary objects.

Parallel uploads work with both service account credentials and Workload Identity authentication. Keep the authentication configuration already defined for your GCS storage. For Workload Identity configuration, see [Workload Identity authentication](workload-identity-auth.md).


### Configure parallel uploads

Set `clientType` to `grpc` and `parallelUploadConcurrency` to a value greater than 1:


```yaml
storage:
  type: gcs
  gcs:
    bucket: <bucket-name>
    prefix: <optional-prefix>
    clientType: grpc
    parallelUploadConcurrency: 4
    chunkSize: 16MB
```

Keep your existing `credentials` section in the configuration file.

Apply the configuration:

```bash
pbm config --file pbm_config.yaml
```
Check the active configuration:

```bash
pbm config --list
```

### Configuration options

| **Option** | **Description**| **Default**|
|------------|----------------|-------------|
| `clientType`               | GCS client used by PBM. Parallel uploads require `grpc`. If you use `json`, PBM performs a standard upload. | `json`                       |
| `parallelUploadConcurrency`| Maximum number of parts PBM uploads concurrently. A value greater than 1 enables parallel uploads. | Parallel uploads are disabled |
| `chunkSize`                | Size of each part uploaded in parallel.                                    | 16 MiB when parallel uploads are enabled |

For the complete list of GCS settings, see [Remote backup storage options](../reference/configuration-options.md).

### Tune upload performance

The `parallelUploadConcurrency` value controls how many parts PBM uploads at the same time. The example uses a value of `4`.

Run a representative backup and compare its duration with a standard upload. Parallel uploads can improve throughput when network and disk speed are not limiting factors.

The `chunkSize` option controls the size of each part. If you do not set it, PBM uses 16 MiB for parallel uploads.

??? example "Parallel upload performance"
    The following configuration uses a 16 MiB part size and allows four concurrent uploads:

    ```yaml
    storage:
    type: gcs
    gcs:
        bucket: pbm-e2e-tests
        prefix: pbme2etest
        clientType: grpc
        chunkSize: 16777216
        parallelUploadConcurrency: 4
    ```

    The following results compare standard uploads with several `parallelUploadConcurrency` values on two environments.

    **i3en.xlarge with 4 vCPUs and a 39.81 GiB dataset**

    | Configuration | Runs | Average |
    |---|---|---|
    | Standard upload | 11m 42s, 9m 55s, 11m 33s, 15m 50s, 11m 29s, 10m 50s, 10m 44s | 11m 43s |
    | Concurrency 4 | 13m 11s, 10m 12s, 13m 32s, 18m 51s | 13m 57s |
    | Concurrency 20 | 9m 54s, 9m 49s | 9m 52s |
    | Concurrency 40 | 13m 11s | 13m 11s |

    **i3en.3xlarge with 12 vCPUs and a 79.64 GiB dataset**

    | Configuration | Runs | Average |
    |---|---|---|
    | Standard upload | 7m 8s, 9m 26s, 7m 15s | 7m 56s |
    | Concurrency 4 | 9m 32s, 7m 46s, 6m 34s | 7m 57s |
    | Concurrency 10 | 6m 39s, 12m 48s, 6m 45s | 8m 44s |
    | Concurrency 20 | 9m 29s, 8m 8s, 8m 19s | 8m 39s |
    | Concurrency 40 | 6m 32s, 7m 31s, 6m 42s | 6m 55s |

    A higher concurrency value does not always produce a faster backup. In this example, concurrency `20` performed best on the 4-vCPU environment, while concurrency `40` performed best on the 12-vCPU environment.

    !!! note
        Performance varies by environment and workload. Network capacity, available CPUs, storage performance, and backup size can all affect upload speed. Compare several concurrency values with a representative backup before choosing a value for your deployment.

### Disable parallel uploads

To disable parallel uploads, remove `parallelUploadConcurrency` from the configuration or set it to `1`.

You can continue using the `gRPC` client with parallel uploads disabled. You do not need to switch back to the `JSON` client.

### Experimental upstream feature

Parallel upload support in the Google Cloud Storage `Go` client is **experimental**. Its behavior and configuration may change in a future upstream release.

Test parallel uploads with representative backup data before enabling them in production. See the upstream [`ParallelUploadConfig` documentation :octicons:](https://pkg.go.dev/cloud.google.com/go/storage#ParallelUploadConfig){="_blank"}.

## Adjust PBM configuration to use GCS

Starting with version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. If you are upgrading from an earlier version, you need to adjust your PBM configuration as follows:

1. Change the `storage.type` from `s3` to `gcs`.
2. Change the `storage.s3` section to `storage.gcs` and adjust the parameters accordingly. See the [Configuration example](#configuration-example) above. Select the option depending on the authentication method you use.

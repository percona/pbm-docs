# Google Cloud Storage (GCS)

You can use Google Cloud Storage (GCS) as a remote backup storage for Percona Backup for MongoDB. 

!!! admonition ""

    Starting from version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. See how to [adjust your PBM configuration to use GCS](#adjust-pbm-configuration-to-use-gcs) after the upgrade.


PBM supports communication with GCS via the JSON API and XML API. The preferred approach is to use the JSON API with a service account. HMAC keys are mainly useful for compatibility with S3-style APIs.

To use GCS, you need the following:

* [create a service account :octicons-link-external-16:](https://cloud.google.com/iam/docs/service-accounts-create#iam-service-accounts-create-console) 
* add keys for the service account:

    * [add JSNON keys :octicons-link-external-16:](https://cloud.google.com/iam/docs/keys-create-delete#creating) or
    * [add HMAC keys :octicons-link-external-16:](https://cloud.google.com/storage/docs/authentication/managing-hmackeys)

* [create a bucket](#create-a-bucket)
* [add the GCS configuration to PBM](#configuration-example) 

## Create a bucket

1. Install and configure the [gcloud CLI :octicons-link-external-16:](https://cloud.google.com/sdk/docs/install)

2. Create a bucket

    ```{.bash data-prompt="$"}
    $ gcloud storage buckets create my-gcs-bucket --location=US
    ```
      
3. Verify the bucket creation

    ```{.bash data-prompt="$"}
    $ gcloud storage buckets list
    ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

## Configuration example

=== "using JSON keys"

    ```yaml
    storage:
     type: gcs
     gcs:
         bucket: pbm-testing
         chunkSize: 16777216
         prefix: pbm/test
         credentials:
           projectId: <your-google-cloud-project-id-here>
           privateKey: <your-private-key-here>
        retryer:
           backoffInitial: 1
           backoffMax: 30
           backoffMultiplier: 2
    ```

=== "using HMAC keys"

	```yaml
	storage:
	 type: gcs
	 gcs:
		 bucket: pbm-testing
		 chunkSize: 16777216
		 prefix: pbm/test
		 endpointUrl: https://storage.googleapis.com
		 credentials:
		   HMACAccessKey: <your-access-key-id-here>
		   HMACSecret: <your-secret-key-here>
        retryer:
           backoffInitial: 1
           backoffMax: 30
           backoffMultiplier: 2
	```

## Adjust PBM configuration to use GCS

Starting with version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. If you are upgrading from an earlier version, you need to adjust your PBM configuration as follows:

1. Change the `storage.type` from `s3` to `gcs`.
2. Change the `storage.s3` section to `storage.gcs` and adjust the parameters accordingly. See the [Configuration example](#configuration-example) above. Select the option depending on the authentication method you use.

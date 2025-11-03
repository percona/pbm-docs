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

## Adjust PBM configuration to use GCS

Starting with version 2.10.0, PBM uses the Google Cloud SDK instead of AWS SDK. If you are upgrading from an earlier version, you need to adjust your PBM configuration as follows:

1. Change the `storage.type` from `s3` to `gcs`.
2. Change the `storage.s3` section to `storage.gcs` and adjust the parameters accordingly. See the [Configuration example](#configuration-example) above. Select the option depending on the authentication method you use.

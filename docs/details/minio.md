# MinIO and S3-compatible storage

Percona Backup for MongoDB (PBM) works with both AWS S3 and other S3-compatible storage services. We test S3-compatible storage services with PBM using [MinIO :octicons-link-external-16:](https://min.io/).

This document provides an overview of MinIO as the closest S3-compatible storage. To use the native AWS S3 service, see [AWS S3 storage](s3-storage.md).

[Configuration example :material-arrow-down:](#configuration-example){.md-button}

## When to use the minio storage type

Use the `minio` storage type in the following scenarios:

* **S3-compatible storage with compatibility issues**: If you're using S3-compatible storage (such as MinIO, Ceph, or other S3 gateways) and encounter upload or download errors when using the `s3` storage type, switch to the `minio` storage type. Check the PBM logs for error messages indicating upload or download failures.

* **S3-compatible storage requiring specific configuration**: Some S3-compatible storage services may require endpoint configuration or other settings that work better with the `minio` storage type.

!!! tip

    For S3-compatible storage, try the `s3` storage type first. Only switch to `minio` if you encounter compatibility issues or errors in the PBM logs.

## Bucket creation

To create a bucket, do the following.
{.power-number}

1. Install a [MinIO client :octicons-link-external-16:](https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc). After the installation, the `mc` is available for you.

2. Configure the `mc` command line tool with a MinIO Server

    ```bash
    mc alias set myminio http://127.0.0.1:9000 MINIO_ACCESS_KEY MINIO_SECRET_KEY
    ```

3. Create a bucket

    ```bash
    mc mb myminio/my-minio-bucket
    ```

4. Verify the bucket creation

    ```bash
    mc ls myminio
    ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

## Configuration example

!!! important
    
    Percona Backup for MongoDB (PBM) needs its own dedicated S3 bucket exclusively for backup-related files. Ensure that this [bucket is created](#bucket-creation) and managed solely by PBM.

This is the example for the basic configuration of MinIO and other S3-compatible storage services in Percona Backup for MongoDB. You can find [the configuration file template :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

```yaml
storage:
  type: minio
  minio:
    endpoint: localhost:9100
    bucket: pbm-example
    prefix: data/pbm/test
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
```

For the description of configuration options, see [Configuration file options](../reference/configuration-options.md).

## Fine-tune storage configuration

The following sections describe how you can fine-tune your storage configuration: 

* [debug logging](#debug-logging) 
* [upload retries](#upload-retries) 
* [data upload to storage with self-signed TLS certificates](#data-upload-to-storage-with-self-signed-tls-certificates)  
* [multiple endpoints to the same S3 storage](endpoint-map.md) 

### Debug logging

You can enable debug logging for different types of storage requests in Percona Backup for MongoDB. Percona Backup for MongoDB prints log messages in the `pbm logs` output so that you can debug and diagnose storage request issues or failures.

To enable debug logging, set the `storage.minio.debugTrace` option in Percona Backup for MongoDB configuration. This instructs PBM to also print HTTP trace from the MinIO storage in the logs.

## Upload retries 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to S3 storage. Set the `storage.minio.retryer.numMaxRetries` option in Percona Backup for MongoDB configuration.

```yaml
retryer:
  numMaxRetries: 3
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

## Data upload to storage with self-signed TLS certificates

Percona Backup for MongoDB supports data upload to S3-compatible storage service over HTTPS with a self-signed or a private CA certificate. This feature is especially important when you use services like MinIO, Ceph, or internal S3 gateways that don't use certificates signed by public Certificate Authorities (CAs).

If your [storage profiles](../features/multi-storage.md) use services with different certificates, PBM needs to trust all of them. You can provide the certificates in one of two ways:

- Set `SSL_CERT_FILE` to a single PEM file that contains all the certificates.
- Set `SSL_CERT_DIR` to a directory that contains separate PEM files.

If neither variable is set, PBM uses the system root certificates. For details about these environment variables, see [Go certificate loading :octicons-link-external-16:](https://pkg.go.dev/crypto/x509#SystemCertPool){:target="_blank"}.

### Usage example

Configure certificate trust on each host where `pbm-agent` or the PBM CLI runs:
{.power-number}

1. Place the certificates in [PEM :octicons-link-external-16:](https://www.ssl.com/guide/pem-der-crt-and-cer-x-509-encodings-and-conversions/){:target="_blank"} format on each host. 

    - For a single file, this example uses `/etc/ssl/minio-ca.crt`. 
    
    - For separate files, place the certificates for each storage service in a directory, such as `/etc/pbm/certificates`.

2. Set the environment variable to point to your certificate file or directory.

    - For a single certificate file or bundle:

        ```bash
        export SSL_CERT_FILE=/etc/ssl/minio-ca.crt
        ```

    - For a directory containing multiple certificate files:

        ```bash
        export SSL_CERT_DIR=/etc/pbm/certificates
        ```

    These commands set the variable for the current shell and processes started from it, including the PBM CLI.

3. If `pbm-agent` runs as a `systemd` service, add the selected variable to the agent's environment file. A shell `export` does not configure the service environment. See [how to find the agent's environment file](../install/configure-authentication.md#set-the-mongodb-connection-uri-for-pbm-agent).

    For example, add this line to use the certificate directory:

    ```ini
    SSL_CERT_DIR=/etc/pbm/certificates
    ```

    Restart the service to apply the change:

    ```bash
    sudo systemctl restart pbm-agent
    ```

4. Check [PBM logs](../reference/pbm-commands.md#pbm-logs) for successful storage access.

Alternatively, you can turn off the TLS verification of the S3 storage in Percona Backup for MongoDB configuration:

```bash
pbm config --set storage.minio.insecureSkipTLSVerify=True
```

!!! warning 

    Use this option with caution as it might leave a hole for man-in-the-middle attacks.


# AWS S3 storage

Percona Backup for MongoDB (PBM) works with AWS S3 and other S3-compatible storage services. We test PBM with the following services:

* [Amazon Simple Storage Service :octicons-link-external-16:](https://docs.aws.amazon.com/s3/index.html)
* [MinIO :octicons-link-external-16:](https://min.io/)

This document provides overview for the native AWS S3 services. To use MinIO and other S3-compatible storage services, see [S3-compatible storage](minio.md).

[Configuration example :material-arrow-down:](#configuration-example){.md-button}

## Storage bucket creation

To create a bucket, do the following.

1. Install and configure [AWS CLI :octicons-link-external-16:](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

2. Create an S3 bucket

    ```bash
    aws s3api create-bucket --bucket my-s3-bucket --region us-east-1
    ```
  
3. Verify the bucket creation

    ```bash
    aws s3 ls
    ```

After the bucket is created, apply the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

## Configuration example

!!! important
    
    Percona Backup for MongoDB (PBM) needs its own dedicated S3 bucket exclusively for backup-related files. Ensure that this [bucket is created](#storage-bucket-creation) and managed solely by PBM.

This is the example for the basic configuration of AWS S3 storage in Percona Backup for MongoDB. You can find [the configuration file template :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.


```yaml
storage:
  type: s3
  s3:
    region: us-west-2
    bucket: pbm-test-bucket
    prefix: data/pbm/backup
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
    serverSideEncryption:
      sseAlgorithm: aws:kms
      kmsKeyID: <your-kms-key-here>
```

For the description of configuration options, see [Configuration file options](../reference/configuration-options.md).

## Fine-tune storage configuration 

The following sections describe how you can fine-tune your storage configuration: 

* [server-side encryption](#server-side-encryption) 
* [debug logging](#debug-logging) 
* [storage classes](#storage-classes)
* [upload retries](#upload-retries) 
* [multiple endpoints to the same S3 storage](endpoint-map.md) 

### Server-side encryption

Percona Backup for MongoDB supports [server-side encryption](../reference/glossary.md#server-side-encryption) for [S3 buckets](../reference/glossary.md#bucket) with the following encryption types:

* [customer-provided keys stored in AWS KMS (SSE-KMS)](#using-aws-kms-keys-sse-kms)
* [customer-provided keys stored on the client side (SSE-C)](#using-customer-provided-keys-sse-c)
* [Amazon S3 managed encryption keys (SSE-S3)](#using-amazon-s3-managed-keys-sse-s3)

####  Using AWS KMS keys (SSE-KMS)

To use the SSE-KMS encryption, specify the following parameters in the Percona Backup for MongoDB configuration file: 

```yaml
serverSideEncryption:
   kmsKeyID: <kms_key_ID>
   sseAlgorithm: aws:kms
```  

#### Using customer-provided keys (SSE-C)

!!! admonition "Version added: [2.0.1](../release-notes/2.0.1.md)" 

Percona Backup for MongoDB also supports server-side encryption with customer-provided keys that are stored on the client side (SSE-C). Percona Backup for MongoDB provides the encryption keys as part of the requests to the S3 storage. The S3 storage uses them to encrypt/decrypt the data using the `AES-256` encryption algorithm. In such a way you save on subscribing to AWS KMS services and can use the server-side encryption with the S3-compatible storage of your choice.

!!! warning

    1. Enable/disable the server-side encryption only for the empty bucket. Otherwise, Percona Backup for MongoDB fails to save/retrieve objects to/from the storage properly.
    2. S3 storage doesn't manage or store the encryption key. It is your responsibility to track what key was used to encrypt what object in the bucket. If you lose the key, any request for an object without the encryption key fails and you lose the object. 

To use the SSE-C encryption, specify the following parameters in the Percona Backup for MongoDB configuration file:    

```yaml
serverSideEncryption:
  sseCustomerAlgorithm: AES256
  sseCustomerKey: <your_encryption_key>
``` 

#### Using Amazon S3 managed keys (SSE-S3)

!!! admonition "Version added: [2.6.0](../release-notes/2.6.0.md)" 

Percona Backup for MongoDB supports server-side encryption with Amazon S3 managed keys (SSE-S3), the default encryption method in Amazon AWS S3. Amazon S3 encrypts each object with a unique key. As an additional safeguard, it encrypts the key itself with a key that it rotates regularly. Amazon S3 server-side encryption uses 256-bit Advanced Encryption Standard Galois/Counter Mode (AES-GCM) to encrypt all uploaded objects.

To use SSE-S3 encryption, specify the following parameters in the Percona Backup for the MongoDB configuration file:

```yaml
serverSideEncryption:
   sseAlgorithm: AES256
```  

### Debug logging

You can enable debug logging for different types of S3 requests in Percona Backup for MongoDB. Percona Backup for MongoDB prints S3 log messages in the `pbm logs` output so that you can debug and diagnose S3 request issues or failures.

To enable S3 debug logging, set the `storage.s3.debugLogLevels` option in the PBM configuration. Supported values are `Signing`, `Retries`, `Request`, `RequestWithBody`, `Response`, `ResponseWithBody`, `DeprecatedUsage`, `RequestEventMessage`, and `ResponseEventMessage`.

See [AWS S3 storage options](../reference/configuration-options.md#storages3debugloglevels) for details and an example.


### Storage classes 

Percona Backup for MongoDB supports [Amazon S3 storage classes :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/). Knowing your data access patterns, you can set the S3 storage class in Percona Backup for MongoDB configuration. When Percona Backup for MongoDB uploads data to S3, the data is distributed to the corresponding storage class. The support of S3 bucket storage types allows you to effectively manage S3 storage space and costs.

To set the storage class, specify the `storage.s3.storageClass` option in Percona Backup for MongoDB configuration file:

```yaml
storage:
  type: s3
  s3:
    storageClass: INTELLIGENT_TIERING
```

When the option is undefined, the S3 Standard (`STANDARD`) storage type is used.

### Upload retries 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to S3 storage as well as the min and max time to wait for the next retry. Set the options `storage.s3.retryer.numMaxRetries`, `storage.s3.retryer.minRetryDelay` and `storage.s3.retryer.maxRetryDelay` in Percona Backup for MongoDB configuration.

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 30
  maxRetryDelay: 5
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

## Data upload to storage with self-signed TLS certificates

Percona Backup for MongoDB supports data upload to S3-compatible storage service over HTTPS with a self-signed or a private CA certificate. This feature is especially important when you use services like MinIO, Ceph, or internal S3 gateways that don't use certificates signed by public Certificate Authorities (CAs).

Use `SSL_CERT_FILE` for a PEM file containing the certificates PBM needs to trust. The file can contain certificates for more than one storage service. Alternatively, use `SSL_CERT_DIR` for a directory containing separate PEM certificate files. This is useful when [storage profiles](../features/multi-storage.md) connect to services that use different certificates.

If neither variable is set, PBM uses the system root certificates. For details about these environment variables, see [Go certificate loading :octicons-link-external-16:](https://pkg.go.dev/crypto/x509#SystemCertPool){:target="_blank"}.

### Usage example

Configure certificate trust on each host where `pbm-agent` or the PBM CLI runs:

1. Place the certificates in PEM format on each host. For a single file, this example uses `/etc/ssl/minio-ca.crt`. For separate files, place the certificates for each storage service in a directory, such as `/etc/pbm/certificates`.

2. Set the environment variable for the approach you chose.

    For a single certificate file or bundle:

    ```bash
    export SSL_CERT_FILE=/etc/ssl/minio-ca.crt
    ```

    For a directory containing multiple certificate files:

    ```bash
    export SSL_CERT_DIR=/etc/pbm/certificates
    ```

    These commands set the variable for the current shell and processes started from it, including the PBM CLI.

3. If `pbm-agent` runs as a systemd service, add the selected variable to the agent's environment file. A shell `export` does not configure the service environment. See [how to find the agent's environment file](../install/configure-authentication.md#set-the-mongodb-connection-uri-for-pbm-agent).

    For example, add this line to use the certificate directory:

    ```ini
    SSL_CERT_DIR=/etc/pbm/certificates
    ```

    Restart the service to apply the change:

    ```bash
    sudo systemctl restart pbm-agent
    ```

4. Check [PBM logs](../reference/pbm-commands.md#pbm-logs) for successful storage access.


Alternatively, you can disable the TLS verification of the S3 storage in Percona Backup for MongoDB configuration:

```bash
pbm config --set storage.s3.insecureSkipTLSVerify=True
```

!!! warning 

    Use this option with caution as it might leave a hole for man-in-the-middle attacks.


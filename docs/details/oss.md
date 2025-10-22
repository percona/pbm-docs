# Alibaba Cloud Object Storage Service (OSS)

If you operate in Asia-Pacific region or China and/or use the Alibaba Cloud infrastructure, you can use the Alibaba Cloud Object Storage Service (OSS) as a remote backup storage for Percona Backup for MongoDB (PBM). This way you ensure low-latency access to your backups and optimize costs.

To use Alibaba Cloud OSS, you need to have:

* an active Alibaba Cloud account with the Object Storage Service enabled for it. Read more about setting up Alibaba Cloud account in the [official documentation :octicons-link-external-16:](https://www.alibabacloud.com/help/en/account/step-1-register-an-alibaba-cloud-account?spm=a2c63.l28256.0.i0)

* an access to the Resource Access Management (RAM) console and sufficient permissions to create and manage access policies and users. Read more about using RAM with Alibaba Cloud OSS in the [official documentation :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/user-guide/how-oss-works-with-ram)

## Create a bucket

You can create a bucket via the [Alibaba Cloud Management Console :octicons-link-external-16:](https://home.console.aliyun.com/) or via the command line. 

=== ":simple-alibabacloud: via Alibaba Cloud Management Console"

    1. Log in to the Alibaba Cloud Management Console.
    2. Navigate to the Object Storage Service (OSS) section.
    3. Navigate to Buckets and click Create a new bucket.
    4. Specify the bucket name, region, and other settings as needed. Refer to bucket naming conventions
    5. Click **Create**, verify the bucket information and click **Confirm**.

=== ":material-console: via Command Line"

    1. [Install](https://www.alibabacloud.com/help/en/oss/developer-reference/install-ossutil2#DAS) and configure the Alibaba Cloud OSS client. After the installation, the `ossutil` command line tool is available for you.
    2. Specify the region:

        ```{.bash data-prompt="$"}
        $ ossutil config
        ```

        Press Enter until you see the prompt `Please enter Region [cn-hangzhou]:` and specify the desired region.

    3. Create a bucket:

		```{.bash data-prompt="$"}
		$ ossutil mb oss://your-bucket-name
		```

		Replace `your-bucket-name` with the desired name for your bucket.

	4. Verify that the bucket is created:

	 	```{.bash data-prompt="$"}
		$ ossutil ls
		```

After you created a bucket, apply the [necessary permissions](storage-configuration.md#permissions-setup) for the user identified by the access credentials you plan to use with PBM.

## Configure access to Alibaba Cloud OSS for PBM

For PBM to successfully access and operate in Alibaba Cloud OSS, it requires access credentials with the necessary permissions to read and write data to the designated OSS bucket.

Alibaba Cloud OSS supports the following access modes:

* Using the Access Key ID and Access Key secret associated with a RAM user. These are permanent credentials designed for programmatic access. Note that the RAM user must have all required permissions to access the OSS resources assigned to them. 

   Refer to the [Use the AccessKey pair of a RAM user to access OSS resources :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/developer-reference/use-the-accesskey-pair-of-a-ram-user-to-initiate-a-request) chapter for detailed instructions.

* Instead of assigning permissions directly to a RAM user, they can obtain access permissions from a RAM role. A RAM role is a virtual identity to which you can attach different access policies with required permissions. To get these permissions, the RAM user assumes the role. 

  The RAM role is used to grant a temporary access to OSS resources using [Secure Token Service :octicons-link-external-16:](https://www.alibabacloud.com/help/en/ram/product-overview/what-is-sts)

  Refer to the [STS temporary access authorization :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/sts-temporary-access-authorization#section-csx-hvf-vdb) chapter for configuration guidelines.


## Configuration example

Here is an example of a Alibaba Cloud OSS configuration in Percona Backup for MongoDB:

== "using AccessKey pair"

   ```yaml
   storage:
	 type: oss
	 oss:
	   region: eu-central-1
	   bucket: your-bucket-name
	   endpointUrl: https://oss-eu-central-1.aliyuncs.com
	   credentials:
	     accessKeyID: "STS.****************"
	     accessKeySecret:  "3dZn*******************************************"
   ```

== "using a RAM role"

   ```yaml
   storage:
	 type: oss
	 oss:
	   region: eu-central-1
	   bucket: your-bucket-name
	   endpointUrl: https://oss-eu-central-1.aliyuncs.com
	   credentials:
	     accessKeyID: "STS.****************" # Temporary access key ID
	     accessKeySecret:  "3dZn*******************************************" # Temporary access key secret
	     roleArn: acs:ram::1234567890123456:role/db-backup-role  
	     sessionName: pbm-backup-session
   ```

See [Configuration file options](../reference/configuration-options.md) for the description of configuration options.

## Fine-tune storage configuration

The following sections describe how you can fine-tune your storage configuration:

* [Server-side encryption](#server-side-encryption)
* [Upload retries](#upload-retries)
* [multiple endpoints to the same S3 storage](endpoint-map.md) 

### Server-side encryption

Alibaba Cloud OSS provides server-side encryption (SSE) capabilities to protect your data at rest. When you enable SSE, your data is automatically encrypted before being stored and decrypted when you access it.

Percona Backup for MongoDB supports server-side encryption for OSS buckets with the following encryption types:

* [Alibaba Cloud OSS-managed encryption keys (SSE-OSS)](#using-oss-managed-encryption-keys-sse-oss). This type provides basic encryption capabilities. 
* [Customer master keys managed by Alibaba Cloud Key Management Service (SSE-KMS)](#using-customer-master-keys-managed-by-key-management-service-sse-kms). This option provides more control over key management and security and is suitable when you need to use self-managed or user-specified keys to meet security and compliance requirements.

Learn more about server-side encryption and billing options when using it in [Server-side encryption :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/user-guide/server-side-encryption-8) documentation.

#### Prerequisites

The RAM user used for PBM to access the Alibaba Cloud OSS must have the required permissions to use server-side encryption on a bucket. Make sure the RAM policy for this user includes the following actions:

1. Permissions to manage the target bucket.

2. The `PutBucketEncryption` and `GetBucketEncryption` permissions.

3. For SSE-KMS encryption type, the RAM user must also have the following permissions:

   * `kms:Encrypt`
   * `kms:Decrypt`
   * `kms:GenerateDataKey`
   * `kms:DescribeKey`

Read more about managing RAM policies in the following Alibaba Cloud OSS documentation:

* [Create a custom RAM policy](https://www.alibabacloud.com/help/en/ram/user-guide/create-a-custom-policy#task-glf-vwf-xdb)
* [Common examples of RAM policies](https://www.alibabacloud.com/help/en/oss/user-guide/common-examples-of-ram-policies) 
* [Permissions for server-side encryption](https://www.alibabacloud.com/help/en/oss/user-guide/server-side-encryption-8#section-oe2-ypt-1fi)

#### Using OSS-managed encryption keys (SSE-OSS)

Server-side encryption with OSS-managed keys (SSE-OSS) is the default encryption method for Alibaba Cloud OSS. Alibaba Cloud OSS automatically generates  encryption keys for each object. It also creates a master key to encrypt encryption keys. 

To configure PBM to use SSE-OSS, add the following options to the `oss` configuration block:

```yaml
serverSideEncryption:
   sseAlgorithm: AES256
```

#### Using customer master keys managed by Key Management Service (SSE-KMS)

Server-side encryption with customer master keys (CMK) managed by Key Management Service (SSE-KMS) gives you more flexibility over key management and security. 

You have the following options:

* use the default customer master key provided by KMS. OSS creates this key in the KMS platform and uses it to encrypt data 
* generate your own customer master key using the KMS console. OSS uses this specified key to encrypt data.

To configure PBM to use SSE-KMS, add the following options to the `oss` configuration block:

```yaml
serverSideEncryption:
   sseAlgorithm: KMS
   kmsMasterKeyID: your-kms-key-id # when using a custom KMS key
   kmsDataEncryption: AES256
```

### Upload retries 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to Alibaba Cloud OSS as well as the min and max time to wait for the next retry. 

Set the following options in Percona Backup for MongoDB configuration.

```yaml
retryer:
  maxAttempts: 5
  maxBackoff: 30
  baseDelay: 30
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

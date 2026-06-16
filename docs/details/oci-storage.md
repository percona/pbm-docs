# Oracle Cloud Infrastructure Object Storage

Percona Backup for MongoDB (PBM) supports [OCI Object Storage](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)
as a remote backup destination through a dedicated OCI native
driver. PBM connects to OCI Object Storage using one of the following authentication types:

| **Authentication type** | **Use when** |
| --- | --- |
| `userPrincipal` | PBM runs anywhere; authenticates with OCI API signing keys |
| `instancePrincipal` | PBM runs on an OCI Compute instance |
| `okeWorkloadIdentity` | PBM runs inside an OKE enhanced cluster |

For information on identity-based authentication methods, refer to the sections on [Instance Principal](oci-wif.md#instanceprincipal) and [OKE Workload Identity](oci-wif.md#okeworkloadidentity).


## Prerequisites

Before configuring PBM, ensure that you have:

- An active OCI tenancy with at least one subscribed region
- The OCI CLI installed and configured (oci setup config) if you intend to use command-line tools. You can also complete the configuration using the OCI Console.
  See the [OCI CLI documentation](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm) 
  for installation instructions
- An OCI user with permission to create compartments, buckets, dynamic groups, and IAM policies in your tenancy

Initialize the OCI CLI configuration if you have not done so already:

```sh
oci setup config
```

Use your tenancy home region as the default CLI region. If the setup generated a new API key, upload the public key 
to your OCI user:

```sh
cat ~/.oci/oci_api_key_public.pem
```

In the OCI Console, go to **User settings → Tokens and keys → 
API keys → Add API key**, paste the public key, and confirm 
the fingerprint matches `~/.oci/config`.

If needed, update local file permissions:

```bash
oci setup repair-file-permissions --file ~/.oci/config
oci setup repair-file-permissions --file ~/.oci/oci_api_key.pem
```

## Verify region access

Check the regions available to your tenancy:

```bash
oci iam region-subscription list \
  --region <HOME_REGION> \
  --output table
```

!!! note
    The region specified in the configuration must be enabled and subscribed to in your OCI tenancy.

## Set up OCI resources

### Export variables

Set the following variables before running any commands in this 
section. All subsequent commands reference them.

```sh
export HOME_REGION=<your-home-region>     # e.g. us-ashburn-1
export BUCKET_REGION=<your-bucket-region> # e.g. eu-frankfurt-1
export COMPARTMENT_NAME=<your-compartment-name>
export BUCKET_NAME=<your-bucket-name>
```
Retrieve and export the values PBM requires:

```sh
export TENANCY_OCID=$(
  oci iam tenancy get \
    --tenancy-id "$(awk -F= '/^tenancy=/{print $2}' ~/.oci/config)" \
    --region "$HOME_REGION" \
    --query 'data.id' \
    --raw-output
)

export USER_OCID=$(awk -F= '/^user=/{print $2}' ~/.oci/config)
export FINGERPRINT=$(awk -F= '/^fingerprint=/{print $2}' ~/.oci/config)
export KEY_FILE=$(awk -F= '/^key_file=/{print $2}' ~/.oci/config)
export KEY_FILE="${KEY_FILE/#\~/$HOME}"
export NAMESPACE=$(
  oci os ns get \
    --region "$BUCKET_REGION" \
    --query 'data' \
    --raw-output
)

echo "TENANCY_OCID: $TENANCY_OCID"
echo "USER_OCID:    $USER_OCID"
echo "FINGERPRINT:  $FINGERPRINT"
echo "KEY_FILE:     $KEY_FILE"
echo "NAMESPACE:    $NAMESPACE"
```
Verify all five values are populated before continuing. An empty value means the OCI CLI is not authenticated or the variable was not set correctly.

### Create a compartment

!!! tip "Important"
    While OCI resources always require a compartment, creating a dedicated one for PBM backup resources is optional. You can use the root compartment. However, Percona recommends using a separate compartment for PBM resources to improve organization and simplify IAM policy management.

Create a compartment for PBM backup resources:

```sh
oci iam compartment create \
  --region "$HOME_REGION" \
  --compartment-id "$TENANCY_OCID" \
  --name "$COMPARTMENT_NAME" \
  --description "PBM OCI Object Storage backup"
```
Wait until the compartment is active, then export its OCID:

```sh
export COMPARTMENT_OCID=$(
  oci iam compartment list \
    --region "$HOME_REGION" \
    --compartment-id "$TENANCY_OCID" \
    --compartment-id-in-subtree true \
    --all \
    --query "data[?name=='$COMPARTMENT_NAME' && \"lifecycle-state\"=='ACTIVE'].id | [0]" \
    --raw-output
)

echo "COMPARTMENT_OCID: $COMPARTMENT_OCID"
```
### Create an Object Storage bucket

Create the bucket:

```sh
oci os bucket create \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --compartment-id "$COMPARTMENT_OCID" \
  --name "$BUCKET_NAME"
```

Verify the bucket was created:

```sh
oci os bucket get \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --bucket-name "$BUCKET_NAME" \
  --query 'data.{name:name,namespace:namespace}' \
  --output table
```

### Create IAM policies

PBM must be able to create, read, overwrite, and delete backup objects.

Ensure your OCI user (or the group it belongs to) has permissions to manage objects in the target bucket/compartment.

Additionally, PBM requires a native copy policy that grants the OCI Object Storage service permission to copy objects internally (for server-side copy operations):

```sh
oci iam policy create \
  --region "$HOME_REGION" \
  --compartment-id "$TENANCY_OCID" \
  --name "pbm-native-copy-$BUCKET_REGION" \
  --description "Allow Object Storage service to copy PBM objects" \
  --statements "[\"Allow service objectstorage-$BUCKET_REGION to manage object-family \
    in compartment $COMPARTMENT_NAME where any { \
      request.permission='OBJECT_READ', \
      request.permission='OBJECT_INSPECT', \
      request.permission='OBJECT_CREATE', \
      request.permission='OBJECT_OVERWRITE', \
      request.permission='OBJECT_DELETE'}\"]"
```

!!! note
    IAM policy changes can take up to 2 minutes to propagate. If PBM reports an authorization error immediately after creating the policies, wait 2 minutes and retry.

## Configure PBM

### userPrincipal

Use this when PBM runs outside OCI, or when you want to 
authenticate with OCI API signing keys.

Generate the correctly indented private key before creating 
the config file:

```sh
sed 's/^/          /' "$KEY_FILE"
```
Create the configuration file:

```yaml
storage:
  type: oci
  oci:
    region: <BUCKET_REGION>
    namespace: <NAMESPACE>
    bucket: <BUCKET_NAME>
    prefix: pbm
    credentials:
      type: userPrincipal
      userPrincipal:
        tenancy: <TENANCY_OCID>
        user: <USER_OCID>
        fingerprint: <FINGERPRINT>
        privateKey: |
          -----BEGIN PRIVATE KEY-----
          ...
          -----END PRIVATE KEY-----
```
!!! warning
    The `user` value must be a user OCID starting with 
    `ocid1.user.oc1`. A bucket or compartment OCID causes 
    a 401 authentication failure.

!!! tip
    Indent the private key correctly before adding it to the configuration:
    
    ```sh
    sed 's/^/          /' "$KEY_FILE"
    ```

## Apply the PBM configuration

Apply the configuration:

```bash
pbm config --file /path/to/oci-config.yaml
```

Force PBM to resync the storage configuration:

```sh
pbm config --force-resync
```

## Verify the configuration

Verify all agents connected and storage initialized successfully:

```sh
pbm status
```

??? example "Output"

    ```sh
    $ pbm status
    Cluster:
    ========
    rs1:
    - rs101:27017 [S]: pbm-agent [v2.15.0] OK
    - rs102:27017 [S]: pbm-agent [v2.15.0] OK
    - rs103:27017 [P]: pbm-agent [v2.15.0] OK

    PITR incremental backup:
    ========================
    Status [OFF]

    Currently running:
    ==================
    (none)

    Backups:
    ========
    Main storage:
    Type:       OCI
    Region:     us-ashburn-1
    Path:       oci://<NAMESPACE>/<BUCKET_NAME>/pbm
    (no snapshots or PITR chunks)
    ```

Every node must show `pbm-agent` as `OK` and storage as `ok`. 

Run a test backup to confirm end-to-end functionality:

```sh
pbm backup
pbm list
```

??? example "Output"

    ```sh
    $ pbm backup
    Starting backup "2026-06-12T07:11:31Z".....
    Backup "2026-06-12T07:11:31Z" saved to remote store (path: "oci://<NAMESPACE>/<BUCKET_NAME>/pbm")
    ```

    ```sh
    $ pbm list
    Backup snapshots:
    NAME   TYPE   PROFILE SELECTIVE   BASE   RESTORE TIME
    -----------------------------------------------------
    2026-06-11T13:14:51Z logical no no  2026-06-11T13:15:07
    2026-06-12T07:04:27Z logical no no  2026-06-12T07:04:42
    2026-06-12T07:11:31Z logical no no  2026-06-12T07:11:46

    PITR <off>:
    ```

## Server-side encryption

PBM supports OCI Object Storage server-side encryption using either OCI Key Management Service (KMS) keys or customer-provided encryption keys (SSE-C). These methods are mutually exclusive. Configure only one encryption mode at a time.

### SSE with OCI KMS

Use OCI KMS encryption when you want OCI to manage encryption and key lifecycle through OCI Vault and Key Management.

#### Create a KMS key

In the OCI Console, navigate to:

**Identity & Security → Key Management & Secret Management → Vault**

Open an existing vault or create a new vault in `$BUCKET_REGION`, then select:

**Master Encryption Keys → Create Key**

Configure the key with the following settings:

- Algorithm: AES
- Key length: 256 bits

Copy the generated key OCID.

Export the key OCID and OCI IAM group name:

```sh
export KMS_KEY_OCID="<KMS_KEY_OCID>"
export OCI_GROUP_NAME="Administrators"
export KMS_POLICY_NAME="pbm-oci-kms-$BUCKET_REGION"
```

If your tenancy uses identity domains, specify the group name in identity-domain-qualified format:

```sh
export OCI_GROUP_NAME="'Default'/'<your-oci-group-name>'"
```

#### Create IAM policies for KMS access

Create a policy that allows both the PBM user group and the regional Object Storage service to use the KMS key.

The object access statement is only required if the OCI group does not already have Object Storage permissions in the target compartment.

```sh
export KMS_OBJECT_POLICY_STATEMENT="Allow group $OCI_GROUP_NAME to manage object-family in compartment $COMPARTMENT_NAME"
export KMS_USER_POLICY_STATEMENT="Allow group $OCI_GROUP_NAME to use keys in compartment $COMPARTMENT_NAME"
export KMS_OBJECTSTORAGE_POLICY_STATEMENT="Allow service objectstorage-$BUCKET_REGION to use keys in compartment $COMPARTMENT_NAME"

oci iam policy create \
  --region "$HOME_REGION" \
  --compartment-id "$TENANCY_OCID" \
  --name "$KMS_POLICY_NAME" \
  --description "Allow PBM user principal and Object Storage to use KMS keys in $BUCKET_REGION" \
  --statements "[\"$KMS_OBJECT_POLICY_STATEMENT\",\"$KMS_USER_POLICY_STATEMENT\",\"$KMS_OBJECTSTORAGE_POLICY_STATEMENT\"]"
```

!!! note
    Wait 1–2 minutes for OCI IAM policy propagation before applying the PBM configuration.

    Without the Object Storage service policy, PBM uploads fail with `NotAuthorizedOrFoundKmsKey` during `CreateMultipartUpload` operations.

#### Configure PBM

Add the following to the OCI storage configuration:

```yaml
serverSideEncryption:
  kmsKeyID: <KMS_KEY_OCID>
```

!!! warning
    Do not configure `sseCustomerKey` together with `kmsKeyID`. PBM treats OCI KMS encryption and SSE-C as mutually exclusive options.

### SSE-C (Customer-Provided Encryption Keys)

Use SSE-C when you want full control of the encryption key material used to encrypt backup objects. PBM sends the encryption key with every Object Storage request. OCI uses the supplied key to encrypt and decrypt objects but does not persist the key.

#### Generate an SSE-C key

Generate a 256-bit AES key and encode it in Base64:

```sh
openssl rand 32 | base64
```

Store the generated value securely. PBM requires the same key for all future backup, restore, and object access operations.

#### Configure PBM

Add the customer-provided key to the storage configuration:

```yaml
serverSideEncryption:
  sseCustomerKey: <BASE64_ENCODED_AES256_KEY>
```

#### Verify encryption

After a successful backup, verify that uploaded objects are encrypted with SSE-C:

```sh
oci os object head \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --bucket-name "$BUCKET_NAME" \
  --name "<object-name>"
```

The response indicates that the object uses customer-supplied encryption keys.

!!! warning
    OCI does not retain SSE-C keys. If the key is lost, encrypted backup objects cannot be decrypted or restored.
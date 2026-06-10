# Configure Oracle Cloud Infrastructure Object Storage

Percona Backup for MongoDB (PBM) can store backups in Oracle Cloud Infrastructure (OCI) Object Storage using the native OCI storage backend.

OCI Object Storage is a cloud object storage service that stores data as objects in buckets. PBM uses this storage to save backup data, metadata, and Point-in-Time Recovery oplog chunks. You can configure PBM to authenticate to OCI by using either a user principal or an instance principal.

## Before you start

Ensure you have the following:

* An Oracle Cloud Infrastructure account
* OCI CLI installed
* Access to create compartments, buckets, and IAM policies
* A running PBM deployment
* The OCI region where you want to store backups
* Access to the OCI user or compute instance that PBM will use

If the bucket region is different from the tenancy home region, subscribe the tenancy to the bucket region first.

## Configure OCI CLI

Initialize the OCI CLI configuration:

```bash
oci setup config
```

Use your tenancy home region as the default CLI region.

If needed, repair local file permissions:

```bash
oci setup repair-file-permissions --file ~/.oci/config
oci setup repair-file-permissions --file ~/.oci/oci_api_key.pem
```

If the setup generated a new API key, upload the public key to your OCI user:

```bash
cat ~/.oci/oci_api_key_public.pem
```

In the OCI Console, go to **User settings** → **Tokens and keys** → **API keys** → **Add API key**, and paste the public key.

## Verify region access

Check the regions available to your tenancy:

```bash
oci iam region-subscription list \
  --region <HOME_REGION> \
  --output table
```

If the bucket region is not listed, subscribe to it in the OCI Console. Open the region menu, select **Manage regions**, find the target region, and click **Subscribe**.

Wait until the subscription is active before creating Object Storage resources.

## Get required OCI values

Export the values required for the PBM configuration:

```bash
export HOME_REGION=<HOME_REGION>
export BUCKET_REGION=<BUCKET_REGION>
export COMPARTMENT_NAME=<COMPARTMENT_NAME>
export BUCKET_NAME=<BUCKET_NAME>
export PBM_PREFIX=pbm
```

Get the tenancy OCID, user OCID, API key fingerprint, private key path, and Object Storage namespace:

```bash
export TENANCY_OCID=$(oci iam tenancy get \
  --tenancy-id "$(awk -F= '/^tenancy=/{print $2}' ~/.oci/config)" \
  --region "$HOME_REGION" \
  --query 'data.id' \
  --raw-output)

export USER_OCID=$(awk -F= '/^user=/{print $2}' ~/.oci/config)
export FINGERPRINT=$(awk -F= '/^fingerprint=/{print $2}' ~/.oci/config)
export KEY_FILE=$(awk -F= '/^key_file=/{print $2}' ~/.oci/config | sed "s|^~|$HOME|")
export NAMESPACE=$(oci os ns get \
  --region "$BUCKET_REGION" \
  --query 'data' \
  --raw-output)
```

## Create a compartment

Create a compartment for PBM backup resources:

```bash
oci iam compartment create \
  --region "$HOME_REGION" \
  --compartment-id "$TENANCY_OCID" \
  --name "$COMPARTMENT_NAME" \
  --description "PBM OCI Object Storage"
```

Export the compartment OCID:

```bash
export COMPARTMENT_OCID=$(oci iam compartment list \
  --region "$HOME_REGION" \
  --compartment-id "$TENANCY_OCID" \
  --compartment-id-in-subtree true \
  --all \
  --query "data[?name=='$COMPARTMENT_NAME' && \"lifecycle-state\"=='ACTIVE'].id | [0]" \
  --raw-output)
```

## Create an Object Storage bucket

Create the bucket:

```bash
oci os bucket create \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --compartment-id "$COMPARTMENT_OCID" \
  --name "$BUCKET_NAME"
```

Verify the bucket:

```bash
oci os bucket get \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --bucket-name "$BUCKET_NAME"
```

## Configure IAM policies

PBM must be able to create, read, overwrite, and delete backup objects.

Create a policy that allows the PBM user group to manage objects in the bucket:

```text
Allow group <OCI_GROUP_NAME> to manage object-family in compartment <COMPARTMENT_NAME> where target.bucket.name = '<BUCKET_NAME>'
```

PBM also uses OCI native server-side copy operations. Add a policy for the regional Object Storage service:

```text
Allow service objectstorage-<BUCKET_REGION> to manage object-family in compartment <COMPARTMENT_NAME> where any {
  request.permission='OBJECT_READ',
  request.permission='OBJECT_INSPECT',
  request.permission='OBJECT_CREATE',
  request.permission='OBJECT_OVERWRITE',
  request.permission='OBJECT_DELETE'
}
```

Allow a few minutes for IAM policy changes to propagate.

## Configure PBM with a user principal

Use user principal authentication when PBM runs outside OCI or when you want to authenticate with OCI API signing keys.

Create a PBM configuration file:

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

Indent the private key correctly before adding it to the configuration:

```bash
sed 's/^/          /' "$KEY_FILE"
```

## Configure PBM with an instance principal

Use instance principal authentication when PBM runs on an OCI Compute instance. This method avoids storing API signing keys in the PBM configuration.

Create a dynamic group that includes the compute instance:

```text
ANY {instance.id = '<INSTANCE_OCID>'}
```

Create a policy that allows the dynamic group to access the bucket:

```text
Allow dynamic-group <DYNAMIC_GROUP_NAME> to manage objects in compartment <COMPARTMENT_NAME> where target.bucket.name = '<BUCKET_NAME>'
```

Then configure PBM:

```yaml
storage:
  type: oci
  oci:
    region: <BUCKET_REGION>
    namespace: <NAMESPACE>
    bucket: <BUCKET_NAME>
    prefix: pbm
    credentials:
      type: instancePrincipal
```

Wait several minutes for IAM policy propagation before testing the configuration.

## Configure server-side encryption with OCI KMS

You can use an OCI Vault key for server-side encryption.

Create or select a Vault key in the same region as the bucket. Then allow both the PBM principal and the Object Storage service to use the key:

```text
Allow group <OCI_GROUP_NAME> to use keys in compartment <COMPARTMENT_NAME>
Allow service objectstorage-<BUCKET_REGION> to use keys in compartment <COMPARTMENT_NAME>
```

Add the key OCID to the PBM configuration:

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
    serverSideEncryption:
      kmsKeyID: <KMS_KEY_OCID>
```

Do not configure `kmsKeyID` and `sseCustomerKey` together. PBM treats OCI KMS and customer-provided encryption keys as mutually exclusive.

## Apply the PBM configuration

Apply the configuration:

```bash
pbm config --file /path/to/oci-config.yaml
```

Force PBM to resync the storage configuration:

```bash
pbm config --force-resync --wait
```

## Verify the configuration

Check that PBM created objects under the configured prefix:

```bash
oci os object list \
  --region "$BUCKET_REGION" \
  --namespace-name "$NAMESPACE" \
  --bucket-name "$BUCKET_NAME" \
  --prefix "$PBM_PREFIX/"
```

Run a backup:

```bash
pbm backup
```

Verify that backup files appear in the OCI bucket.


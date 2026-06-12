# Oracle Cloud Infrastructure Object Storage

Percona Backup for MongoDB (PBM) supports [OCI Object Storage](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm) as a remote backup destination through a dedicated OCI native driver, allowing you to store and manage MongoDB backups directly in OCI buckets.

PBM connects to OCI Object Storage using the `userPrincipal` authentication type by default, which uses OCI API signing keys. For keyless authentication, PBM also supports `instancePrincipal` for PBM running on OCI Compute instances.

## Prerequisites

Before configuring PBM, ensure that you have:

- An Oracle Cloud Infrastructure account
- An Object Storage bucket
- A user with access to the bucket
- An OCI API signing key pair (private key + uploaded public key) for the user
- Bucket permissions that allow reading and writing objects


## Configure OCI CLI

Initialize the OCI CLI configuration:

```sh
oci setup config
```

Use your tenancy home region as the default CLI region.

If needed, update local file permissions:

```bash
oci setup repair-file-permissions --file ~/.oci/config
oci setup repair-file-permissions --file ~/.oci/oci_api_key.pem
```
If the setup generated a new API key, upload the public key to your OCI user:

```bash
cat ~/.oci/oci_api_key_public.pem
```

In the OCI Console, go to **User settings → Tokens and keys → API keys → Add API key**, and paste the public key.

## Verify region access

Check the regions available to your tenancy:

```bash
oci iam region-subscription list \
  --region <HOME_REGION> \
  --output table
```

!!! note
    The region specified in the configuration must be enabled and subscribed to in your OCI tenancy.

## Get required OCI values

Export the values required for the PBM configuration:

```bash
export HOME_REGION=<HOME_REGION>
export BUCKET_REGION=<BUCKET_REGION>
export COMPARTMENT_NAME=<COMPARTMENT_NAME>
export BUCKET_NAME=<BUCKET_NAME>
export PBM_PREFIX=pbm
```

Get the `tenancy OCID`, user `OCID`, `API key fingerprint`, `private key path`, and `Object Storage namespace`: 

```bash
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

export NAMESPACE=$(
  oci os ns get \
    --region "$BUCKET_REGION" \
    --query 'data' \
    --raw-output
)
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

Allow group `<OCI_GROUP_NAME>` to manage object-family in compartment `<COMPARTMENT_NAME>`

PBM also uses OCI native server-side copy operations. Add a policy for the regional Object Storage service:

```bash
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

!!! tip
    Indent the private key correctly before adding it to the configuration:
    
    ```sh
    sed 's/^/          /' "$KEY_FILE"
    ```

## Configure PBM with an instance principal

Use instance principal authentication when PBM runs on an OCI Compute instance. This method avoids storing API signing keys in the PBM configuration.

Create a dynamic group that includes the compute instance:

```sh
ANY {instance.id = '<INSTANCE_OCID>'}
```

Create a policy that allows the dynamic group to access the bucket:

```sh
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
Wait for a few minutes for IAM policy propagation before testing the configuration.

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

Run a backup:

```bash
pbm backup
```

Check that PBM created objects under the configured prefix:

```bash
oci os object list \
  --bucket-name <BUCKET_NAME> \
  --prefix pbm/
```

You should see backup metadata and backup files stored under the configured prefix.
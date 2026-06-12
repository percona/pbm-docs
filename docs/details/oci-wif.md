# Oracle Cloud Infrastructure (OCI) Object Storage with Workload Identity Authentication

Percona Backup for MongoDB (PBM) supports multiple authentication methods for Oracle Cloud Infrastructure (OCI) Object Storage, including Workload Identity. With Workload Identity, PBM can access OCI resources without storing or managing API keys, reducing operational overhead and improving security.

Percona Backup for MongoDB (PBM) supports the default `userPrincipal` authentication method as well as the following Workload Identity options:

| Auth type | When to use |
|---|---|
|`userPrincipal`|PBM is running anywhere — on-premises, on other clouds, or on OCI|
| `instancePrincipal`| PBM is running on a virtual machine inside OCI |
| `okeWorkloadIdentity`| PBM is running inside an OKE enhanced cluster|

## userPrincipal

Choose `userPrincipal` when PBM runs outside OCI, or when you want a single authentication approach that works in any environment. PBM authenticates using an OCI user account and an API signing key.

### Before you begin

You need:

- An OCI user account with access to the target bucket
- An [API signing key pair :octicons-link-external-16:](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm){:target="_blank"} (private key file and its fingerprint)
- The [Oracle Cloud Identifier (OCID) :octicons-link-external-16:](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm#Oracle){:target="_blank"} of the user and tenancy
- The name of the OCI bucket PBM will use for backups

### Procedure

1. **Create an IAM policy**

    Grant the user permission to manage objects in the target bucket:

    ```sh
    oci iam policy create \
        --region "$HOME_REGION" \
        --compartment-id "$TENANCY_OCID" \
        --name "$USER_POLICY_NAME" \
        --description "Allow PBM user to access $BUCKET_NAME" \
        --statements "[\"Allow group $USER_GROUP_NAME to manage objects in compartment $COMPARTMENT_NAME where target.bucket.name = '$BUCKET_NAME'\"]"
    ```
  
    Replace the following variables:

    | Variable | Description |
    |---|---|
    | `HOME_REGION` | Your tenancy's home region (e.g. `us-ashburn-1`) |
    | `TENANCY_OCID` | OCID of your OCI tenancy |
    | `USER_POLICY_NAME` | A name for the policy (e.g. `pbm-user-policy`) |
    | `USER_GROUP_NAME` | The IAM group the PBM user belongs to |
    | `COMPARTMENT_NAME` | Name of the compartment containing the bucket |
    | `BUCKET_NAME` | Name of the OCI Object Storage bucket |

2. **Configure PBM authentication**

    In your PBM configuration, set the storage type to `oci` and the credentials type to `userPrincipal`. Provide the API signing key private key in PEM format.

    ```yaml
    storage:
      type: oci
      oci:
        region: <bucket_region>
        namespace: <namespace>
        bucket: <bucket_name>
        prefix: <path_prefix>
        credentials:
          type: userPrincipal
          userPrincipal:
            tenancy: <tenancy_ocid>
            user: <user_ocid>
            fingerprint: <key_fingerprint>
            privateKey: |
              -----BEGIN PRIVATE KEY-----
              ...
              -----END PRIVATE KEY-----
    ```

## instancePrincipal

Choose `instancePrincipal` when PBM runs directly on an OCI Compute instance. PBM automatically obtains OCI credentials from the instance, eliminating the need for credential files or API keys.

### Before you begin

You need:

- The [Oracle Cloud Identifier (OCID) :octicons-link-external-16:](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm#Oracle){:target="_blank"} of the OCI Compute instance running PBM
- The name of the OCI bucket PBM will use for backups.

### Procedure

Follow these steps to set up OCI using **`instancePrincipal`:**

1. **Create a dynamic group for the instance**

    OCI IAM policies cannot target individual instances directly. You must first add the instance to a dynamic group, then write a policy against that group.

    ```sh
    oci iam dynamic-group create \
    --region "$HOME_REGION" \
    --compartment-id "$TENANCY_OCID" \
    --name "$INSTANCE_DYNAMIC_GROUP_NAME" \
    --description "PBM VM instance principal" \
    --matching-rule "ANY {instance.id = '$INSTANCE_OCID'}"
    ```

    Replace the following variables:

    | Variable | Description |
    |---|---|
    | `HOME_REGION` | Your tenancy's home region (e.g. `us-ashburn-1`) |
    | `TENANCY_OCID` | OCID of your OCI tenancy |
    | `INSTANCE_DYNAMIC_GROUP_NAME` | A name for the dynamic group (e.g. `pbm-vm-group`) |
    | `INSTANCE_OCID` | OCID of the Compute instance running PBM |

2. **Create an IAM policy**

    Grant the dynamic group permission to manage objects in the target bucket:

    ```sh
    export INSTANCE_POLICY_STATEMENT="Allow dynamic-group $INSTANCE_DYNAMIC_GROUP_NAME \
    to manage objects in compartment $COMPARTMENT_NAME \
    where target.bucket.name = '$BUCKET_NAME'"

    oci iam policy create \
    --region "$HOME_REGION" \
    --compartment-id "$TENANCY_OCID" \
    --name "$INSTANCE_POLICY_NAME" \
    --description "Allow PBM VM instance principal to access $BUCKET_NAME" \
    --statements "[\"$INSTANCE_POLICY_STATEMENT\"]"
    ```

    Replace the following additional variables:

    | Variable | Description |
    |---|---|
    | `COMPARTMENT_NAME` | Name of the compartment containing the bucket |
    | `BUCKET_NAME` | Name of the OCI Object Storage bucket |
    | `INSTANCE_POLICY_NAME` | A name for the policy (e.g. `pbm-vm-policy`) |

3. **Configure PBM authentication**

    In your PBM configuration, set the storage type to `oci` and the credentials type to `instancePrincipal`. No key file or passphrase is needed.

    ```yaml
    storage:
      type: oci
      oci:
        region: <bucket_region>
        namespace: <namespace>
        bucket: <bucket_name>
        prefix: <path_prefix>
        credentials:
          type: instancePrincipal
    ```

## okeWorkloadIdentity

Choose `okeWorkloadIdentity` when PBM runs as a workload in an Oracle Kubernetes Engine (OKE) enhanced cluster. The Kubernetes service account token is exchanged for OCI credentials automatically by the OKE Workload Identity service.

!!! note
    Your OKE cluster must be an **enhanced cluster** with Workload Identity enabled. Basic clusters do not support this feature.

In your PBM configuration, set the storage type to `oci` and the credentials type to `okeWorkloadIdentity`:

```yaml
storage:
  type: oci
  oci:
    region: <bucket_region>
    namespace: <namespace>
    bucket: <bucket_name>
    prefix: <path_prefix>
    credentials:
      type: okeWorkloadIdentity
```

For setup instructions, see [Configure OKE Workload Identity for workloads :octicons-link-external-16:](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contenggrantingworkloadaccesstoresources.htm){:target="_blank"}.
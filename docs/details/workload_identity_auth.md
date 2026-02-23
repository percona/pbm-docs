# Workload Identity authentication for Google Cloud Storage

!!! info "Important"
    Workload Identity Federation (WIF) for Google Cloud Storage 
    (GCS) is supported in PBM version **2.13.0 or later**.

Percona Backup for MongoDB (PBM) now supports Workload Identity Federation (WIF) for authenticating with Google Cloud Storage (GCS).

This feature enables secure backup uploads without relying on static service account JSON keys. Instead, PBM uses short-lived, **automatically refreshed tokens** obtained through federation with an external identity provider (IdP).

Workload Identity Federation lets on‑premises or multicloud workloads access Google Cloud resources using federated identities instead of a service account key, eliminating the maintenance and security burden of service account keys.


!!! info "Important"
    The exact configuration steps depend on where PBM runs (GCE VM, GKE, on-prem, AWS, Azure, GitHub Actions, etc.). This section explains what PBM needs, and provides one end-to-end example for GCE VM, which is the simplest setup.

## How it works with PBM

PBM integrates with Workload Identity Federation as follows:
{.power-number}

1. PBM authenticates with its external IdP (e.g., OIDC, SAML, AWS, Azure).

2. PBM exchanges the IdP credential with Google’s Security Token Service (STS).

3. STS issues a short-lived federated token.

4. PBM uses this token to impersonate a Google Cloud service account with the required GCS permissions. PBM communicates with GCS using Google Cloud libraries/SDKs (PBM 2.10.0+ uses the Google Cloud SDK for GCS).

5. Backups are uploaded securely to GCS without static keys.

With Workload Identity Authentication, PBM relies on **Application Default Credentials** (ADC) provided by the runtime (for example, GKE metadata server, or an external Workload Identity Federation credential configuration file). When ADC is available, PBM can upload and download backups from GCS **without embedding JSON private keys** in the PBM config.

## Prerequisites

To use Workload Identity with GCS, you must have the following:
{.power-number}

1. Your runtime environment must provide ADC (for example:
    - a GCE VM or GKE node/pod with Google identity available via metadata server, or

    - an external WIF credential configuration file referenced via GOOGLE_APPLICATION_CREDENTIALS).

2. The Google service account (GSA) that PBM uses must have the required GCS permissions on the target bucket.

3. Enable Workload Identity in PBM’s GCS storage config:

    ```yaml
    storage:
    type: gcs
    gcs:
        bucket: <YOUR_BUCKET_NAME>
        prefix: <YOUR_PREFIX>
        credentials:
        workloadIdentity: true
    ```
    PBM will then use the ADC credentials provided by the environment (rather than a static JSON private key).


## Usecase: GCE Virtual Machine (simplest path)

On a GCE VM, the **Workload Identity** is just attaching a GSA to the VM and letting applications use ADC from the metadata server.
{.power-number}

1. Create a bucket.

    ```bash
    gcloud storage buckets create gs://<BUCKET_NAME> --location=<REGION>
    ```

2. Create a Google service account (GSA).

    ```bash
    gcloud storage buckets create gs://<BUCKET_NAME> --location=<REGION>
    ```

3. Grant the GSA permissions on the bucket.
    
    For example, grant object read/write (choose the role that matches your needs/policy):

    ```bash
    gcloud storage buckets add-iam-policy-binding gs://<BUCKET_NAME> \
  --member="serviceAccount:<GSA_NAME>@<PROJECT_ID>.iam.gserviceaccount.com" \
  --role="roles/storage.objectUser"
  ```

4. Create a VM with the GSA attached.

5. Use the PBM config snippet below (note the workloadIdentity: `true` flag, and no `JSON` key):

    ```yaml
    storage:
      type: gcs
      gcs:
        bucket: <BUCKET_NAME>
        prefix: pbm
        credentials:
          workloadIdentity: true
    ```

6. Apply the config.

    ```bash
    pbm config --file pbm_config.yaml
    ```

7. Verify authentication.

    On the VM, you can sanity check that the identity is available via ADC (for example by listing bucket contents), then run a PBM backup to confirm uploads succeed.
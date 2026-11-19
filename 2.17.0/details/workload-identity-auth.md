> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Workload Identity authentication for Google Cloud Storage

Version added: 2.13.0

Workload Identity Federation (WIF) for Google Cloud Storage (GCS) is supported in PBM version 2.13.0 or later.

Percona Backup for MongoDB (PBM) now supports Workload Identity Federation (WIF) for authenticating with Google Cloud Storage (GCS).

This feature enables secure backup uploads without relying on static service account JSON keys. Instead, PBM uses short-lived, **automatically refreshed tokens** obtained through federation with an external identity provider (IdP).

Workload Identity Federation lets on-premises or multicloud workloads access Google Cloud resources using federated identities instead of a service account key, eliminating the maintenance and security burden of service account keys.

Note

The exact configuration steps depend on where PBM is deployed (GCE VM, GKE, on-prem, AWS, Azure, GitHub Actions, etc.). This section outlines the requirements for PBM and provides a comprehensive end-to-end example for GCE VM, which is the simplest setup.

## How it works with PBM

PBM integrates with Workload Identity Federation as follows:

1. PBM authenticates with its external IdP (e.g., OIDC, SAML, AWS, Azure).
1. PBM exchanges the IdP credential with Google’s Security Token Service (STS).
1. STS issues a short-lived federated token.
1. PBM uses this token to impersonate a Google Cloud service account with the required GCS permissions. PBM communicates with GCS using Google Cloud libraries/SDKs (PBM 2.10.0+ uses the Google Cloud SDK for GCS).
1. Backups are uploaded securely to GCS without static keys.

With Workload Identity Authentication, PBM uses [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials) (ADC) provided by the runtime, such as the GKE metadata server or an external Workload Identity Federation credential configuration file. When ADC is available, PBM can upload and download backups from Google Cloud Storage (GCS) **without the need to embed JSON private keys** in the PBM configuration.

## Prerequisites

To use Workload Identity with GCS, you must have the following:

1. Your runtime environment must provide ADC.

   - A Google Compute Engine (GCE) VM or GKE node/pod with Google identity available via metadata server, or
   - An external WIF credential configuration file referenced via `GOOGLE_APPLICATION_CREDENTIALS`.

1. The Google service account (GSA) that PBM uses must have the required GCS permissions on the target bucket.

1. Enable Workload Identity in PBM’s GCS storage config:

   ```
   storage:
     type: gcs
     gcs:
       bucket: <YOUR_BUCKET_NAME>
       prefix: <YOUR_PREFIX>
       credentials:
         workloadIdentity: true
   ```

   PBM will then use the ADC credentials provided by the environment (rather than a static JSON private key). You can find [the configuration file template](https://github.com/percona/percona-backup-mongodb/blob/v2.17.0/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

## Use case: GCE Virtual Machine (simplest path)

Important

These commands assume the **Google Cloud CLI** (gcloud) is installed and configured on the machine you run them from, and that you have permission to create service accounts, VMs, and update IAM.

On a GCE VM, Workload Identity is implemented by attaching a GSA to the VM and letting applications use ADC from the metadata server.

1. Create a bucket.

   ```
   gcloud storage buckets create gs://<BUCKET_NAME> --location=<REGION>
   ```

1. Create a Google service account (GSA).

   ```
   gcloud iam service-accounts create <GSA_NAME> \
     --project=<PROJECT_ID> \
     --display-name="<GSA_NAME>"
   ```

1. Grant the GSA permissions on the bucket.

   For example, grant object read/write (choose the role that matches your needs/policy):

   ```
   gcloud storage buckets add-iam-policy-binding gs://<BUCKET_NAME> \
    --member="serviceAccount:<GSA_NAME>@<PROJECT_ID>.iam.gserviceaccount.com" \
    --role="roles/storage.objectUser"
   ```

1. Create a VM with the GSA attached.

   ```
   gcloud compute instances create <VM_NAME> \
     --project=<PROJECT_ID> \
     --zone=<ZONE> \
     --service-account=<GSA_NAME>@<PROJECT_ID>.iam.gserviceaccount.com \
     --scopes=cloud-platform
   ```

   5. Use the PBM config snippet below (note the workloadIdentity: `true` flag, and no JSON key):

   ```
   storage:
     type: gcs
     gcs:
       bucket: <BUCKET_NAME>
       prefix: pbm
       credentials:
         workloadIdentity: true
   ```

   Note

   - Application Default Credentials (ADC) are used by PBM only when `credentials.workloadIdentity: true` is set.
   - With `workloadIdentity: true`, PBM relies on standard ADC sources such as the GCE/GKE metadata server or a **Workload Identity Federation external account configuration file** referenced via `GOOGLE_APPLICATION_CREDENTIALS`.
   - When using `workloadIdentity: true`, pbm validates that `GOOGLE_APPLICATION_CREDENTIALS` file is configured for workload federation and will reject any other credentials.

1. Apply the config.

   ```
   pbm config --file pbm_config.yaml
   ```

1. Verify authentication.

   On the VM, you can check that the identity is available through ADC by listing the bucket contents. Then, run a PBM backup to ensure that the uploads succeed.

   ```
   gcloud storage ls gs://<BUCKET_NAME>
   ```

   ```
   pbm backup
   ```

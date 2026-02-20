# Workload Identity authentication for GCS

Percona Backup for MongoDB (PBM) now supports Workload Identity Federation (WIF) for authenticating with Google Cloud Storage (GCS).

This feature enables secure backup uploads without relying on static service account JSON keys. Instead, PBM uses short-lived, **automatically refreshed tokens** obtained through federation with an external identity provider (IdP).

## Why Workload Identity

Workload Identity Federation lets on‑premises or multicloud workloads access Google Cloud resources using federated identities instead of a service account key, eliminating the maintenance and security burden of service account keys.

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

Before running commands, make sure you have:

- A Google Cloud Storage (GCS) bucket created for PBM backups. If you don’t already have a bucket, follow the steps in the main GCS storage documentation: see [Create a bucket](gcs.md#create-a-bucket).

- PBM version 2.10.0 or higher

- A Google Cloud project (you need both):

    - PROJECT_ID (string like my-gcp-project)

    - PROJECT_NUMBER (numeric)

- An **external Identity Provider (IdP)** that can provide identity tokens (commonly OIDC)

- `gcloud` installed and authenticated as an admin who can create IAM resources


## Configuration steps

Follow these steps to configure Workload Identity Federation for PBM:
{.power-number}

1. Set your variables once:

    ```bash
    # Required: your Google Cloud project ID (string)
export PROJECT_ID="my-gcp-project"

  # Recommended: fetch the numeric project number automatically
  export PROJECT_NUMBER="$(gcloud projects describe "$PROJECT_ID" --format="value(projectNumber)")"

  # Workload Identity pool/provider IDs you are creating
  export POOL_ID="pbm-pool"
  export PROVIDER_ID="pbm-provider"

  # Service account PBM will impersonate
  export SA_NAME="pbm-backup-sa"
  export SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"

  # GCS bucket where PBM writes backups
  export BUCKET="my-backup-bucket"

  # The external identity subject from your IdP (must match what your provider maps to google.subject)
  # Example values depend on your IdP (OIDC 'sub' claim is most common).
  export WORKLOAD_SUBJECT="YOUR_WORKLOAD_IDENTITY_SUBJECT"

  # OIDC issuer URL for your IdP
  export ISSUER_URI="https://YOUR-IDP.example.com"

    ```

2. Create a Workload Identity pool:

    ```bash
    gcloud iam workload-identity-pools create "$POOL_ID" \
      --location="global" \
      --display-name="PBM Workload Identity Pool"
    ```

3. Create a provider (OIDC Example):

    This maps the IdP subject (`assertion.sub`) to Google’s `google.subject`.

    The following example uses an OIDC provider (e.g., Kubernetes, GitHub Actions). For AWS, replace `--issuer-uri` with `--aws`.

    ```bash
    gcloud iam workload-identity-pools providers create-oidc pbm-provider \
    --workload-identity-pool="$POOL_ID" \
    --issuer-uri="$ISSUER_URI"\
    --location="global" \
    --attribute-mapping="google.subject=assertion.sub"
    ```

4. Create a service account for PBM backups. This service account will be impersonated by PBM when uploading backups to GCS.

    ```bash
    gcloud iam service-accounts create $SA_NAME \
    --display-name="PBM Backup Service Account"
    ```

5. Grant service account impersonation:

    ```bash
    gcloud iam service-accounts add-iam-policy-binding "$SA_EMAIL" \
      --role="roles/iam.workloadIdentityUser" \
      --member="principal://iam.googleapis.com/projects/$PROJECT_NUMBER/locations/global/workloadIdentityPools/$POOL_ID/subject/$WORKLOAD_SUBJECT"
    ```
    **Where:**

    **PROJECT_ID →** Your Google Cloud project ID (string, e.g., `my-gcp-project`).

    **PROJECT_NUMBER →** The numeric project identifier (find with `gcloud projects describe PROJECT_ID --format="value(projectNumber)"`).

    **WORKLOAD_ID →** The identity subject from your IdP that PBM uses (for example, a Kubernetes service account name or GitHub Actions workflow ID).

    **YOUR-IDP →** The issuer URI of your identity provider, i.e. the value you used for the `--issuer-uri` flag in step 3 (for example, `https://accounts.google.com` for Google, or your OIDC provider URL).

6. Assign GCS permissions:

    ```bash
    gcloud projects add-iam-policy-binding "$PROJECT_ID" \
      --member="serviceAccount:$SA_EMAIL" \
      --role="roles/storage.objectAdmin"
    ```

    Ensure that the bucket has the proper [permissions for PBM to use the bucket](storage-configuration.md#permissions-setup).

7. Generate and configure the Workload Identity credential configuration file:

    For environments that do not provide credentials via a metadata server (for example, on‑premises, GitHub Actions, or other external IdPs), create a Workload Identity Federation credential configuration file and make it available to PBM as Application Default Credentials (ADC).

    1. **Generate the credential configuration file:**

        ```bash
        gcloud iam workload-identity-pools create-cred-config \
          projects/$PROJECT_NUMBER/locations/global/workloadIdentityPools/$POOL_ID/providers/pbm-provider \
          --service-account="$SA_EMAIL" \
          --output-file="pbm-wif-cred.json"
        ```

    2. **Make the file available to PBM:**

        Place `pbm-wif-cred.json` on the host or container where PBM runs, and set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable so that PBM (via Google Cloud SDK) can pick it up as ADC:

        ```bash
        export GOOGLE_APPLICATION_CREDENTIALS="/path/to/pbm-wif-cred.json"
        ```

        Ensure this environment variable is set in the context where PBM commands run (for example, in the PBM container spec, systemd unit, or shell session).

8. PBM configuration:
    When using Workload Identity, omit the credentials block in the PBM configuration. The Google Cloud SDK (used by PBM 2.10+) will automatically detect the environment's identity.

    1. **Create config file:**
        Create a file named `pbm_config.yaml`:

        ```yaml
        storage:
          type: gcs
          gcs:
            bucket: <YOUR_BUCKET_NAME>
            prefix: <YOUR_PREFIX>
            # No credentials block here! 
            # PBM will use the ambient Workload Identity.
        ```

    2. Apply the config:

        ```bash
        pbm config --file pbm_config.yaml
        ```

    ??? example "Example PBM configuration file"
        ```yaml
        storage:
          type: gcs
          gcs:
            bucket: my-backup-bucket
            prefix: pbm
        ```




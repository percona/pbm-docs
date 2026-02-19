# Workload Identity authentication

Percona Backup for MongoDB (PBM) now supports Workload Identity Federation (WIF) for authenticating with Google Cloud Storage (GCS).

This feature enables secure backup uploads without relying on static service account JSON keys. Instead, PBM uses short‑lived, **automatically refreshed tokens** obtained through federation with an external identity provider (IdP).

## Why Workload Identity

Workload Identity Federation lets on‑premises or multicloud workloads access Google Cloud resources using federated identities instead of a service account key, eliminating the maintenance and security burden of service account keys.

## How this works with PBM

This is how Workload Identity Federation Works:
{ .power-number }

1. PBM authenticates with its external IdP (e.g., OIDC, SAML, AWS, Azure).

2. PBM exchanges the IdP credential with Google’s Security Token Service (STS).

3. STS issues a short‑lived federated token.

4. PBM uses this token to impersonate a Google Cloud service account with the required GCS permissions. PBM communicates with GCS using Google Cloud libraries/SDKs (PBM 2.10.0+ uses the Google Cloud SDK for GCS).

5. Backups are uploaded securely to GCS without static keys.

With Workload Identity Authentication, PBM relies on **Application Default Credentials** (ADC) provided by the runtime (for example, GKE metadata server, or an external Workload Identity Federation credential configuration file). When ADC is available, PBM can upload and download backups from GCS **without embedding JSON private keys** in the PBM config.

## Configuration steps

Follow theese steps to configure Workload Identity Federation for PBM:
{ .power-number }

1. Create a Workload Identity pool:

    ```
    gcloud iam workload-identity-pools create pbm-pool \
    --location="global" \
    --display-name="PBM Workload Identity Pool"
    ```
2. Configure a provider (OIDC Example):

The following example uses an OIDC provider (e.g., Kubernetes, GitHub Actions). For AWS, replace `--oidc-issuer-uri` with `--aws`.

    ```
    gcloud iam workload-identity-pools providers create-oidc pbm-provider \
  --workload-identity-pool="pbm-pool" \
  --issuer-uri="https://YOUR-IDP.example.com" \
  --location="global" \
  --attribute-mapping="google.subject=assertion.sub"
    ```

3. Grant service account impersonation:

    ```sh
    gcloud iam service-accounts add-iam-policy-binding \
  pbm-backup-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principal://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/pbm-pool/subject/WORKLOAD_ID"
    ```

4. Assign GCS permissions:

    ```
    gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:pbm-backup-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
    ```

5. PBM configuration:

    When using Workload Identity, you omit the credentials block in the PBM configuration. The Google Cloud SDK (used by PBM 2.10+) will automatically detect the environment's identity.

    1. **New config format (YAML)**
        Create a file named `pbm_config.yaml`:

        ```bash
        storage:
          type: gcs
          gcs:
            bucket: [YOUR_BUCKET_NAME]
            prefix: [YOUR_PREFIX]
            # No credentials block here! 
            # PBM will use the ambient Workload Identity.
        ```

    2. Apply the config:

        ```bash
        pbm config --file pbm_config.yaml
        ```

??? Example "Example PBM configuration snippet"
    ```yaml
    storage:
      type: gcs
      bucket: my-backup-bucket
      auth:
        method: workload-identity
        provider: pbm-provider
        service-account: pbm-backup sa@PROJECT_ID.iam.gserviceaccount.com
    ```

!!! note
    - **PBM version:** Ensure you are using PBM 2.10.0 or higher. Earlier versions used the AWS SDK (S3 compatibility) which required HMAC keys.
    - **ADC (Application Default Credentials):** The PBM agent code calls the Google Cloud storage client. By removing the credentials from the config, the client defaults to google.FindDefaultCredentials().
    - **Environment variables:** If you are using Workload Identity Federation (for on-prem/other clouds), you must set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable on the PBM agent pods/servers to point to the generated `credential-configuration.json` file.



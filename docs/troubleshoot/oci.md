# Troubleshoot issues for OCI

This page lists common Oracle Cloud Infrastructure (OCI) Object Storage issues when using PBM. For configuration instructions, see [Configure Oracle Cloud Infrastructure Object Storage](../details/oci-storage.md).
## Storage is not initialized

If PBM reports that storage is not initialized, verify the following:

* The bucket exists.
* The bucket region is correct.
* The namespace is correct.
* The PBM principal has access to the bucket.
* The Object Storage service copy policy exists.
* IAM policy propagation has completed.

## KMS authorization errors

If uploads fail with `NotAuthorizedOrFoundKmsKey`, verify the following:

* The KMS key exists in the bucket region.
* The PBM principal can use the key.
* The regional Object Storage service can use the key.
* The KMS policy has propagated.

## Instance principal access fails

If instance principal authentication fails, verify the following:

* The compute instance is included in the dynamic group.
* The dynamic group policy grants access to the bucket.
* The instance and bucket are in the expected tenancy and compartment.
* IAM policy propagation has completed.
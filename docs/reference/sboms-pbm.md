# Software Bill of Materials


A Software Bill of Materials (SBOM) is a machine-readable inventory of the components and dependencies included in a software release. It helps you understand what is included in a build and assess potential security or compliance risks.

Starting with version 2.15.0, every Percona Backup for MongoDB (PBM) release includes a [CycloneDX 1.6 :octicons-link-external-16:](https://cyclonedx.org/specification/overview/){:target="_blank"} SBOM in JSON format.

## Why it matters

An SBOM helps you:

- Identify the components and dependencies included in a PCSM release.
- Assess known vulnerabilities using SBOM-compatible security scanners.
- Support security reviews, compliance processes, and software supply chain requirements.
- Verify the contents of deployed software artifacts.

## Where to find the SBOM

| Distribution method | SBOM location |
|---|---|
| Binary tarball | `Percona Backup for MongoDB.cdx.json` at the archive root |
| RPM package | `/usr/share/doc/percona-backup-mongodb/percona-backup-mongodb-<version>.cdx.json` |
| DEB package | `/usr/share/doc/percona-backup-mongodb/percona-backup-mongodb-<version>.cdx.json` |
| Docker image | Two SBOMs ship together — see [Docker images](#docker-images) below |
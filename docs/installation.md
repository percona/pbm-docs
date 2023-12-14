# Quickstart guide

Percona Backup for MongoDB (PBM) is an open source and distributed solution for consistent backups and restore of [MongoDB sharded clusters and replica sets](details/deployments.md). [Learn how PBM works](intro.md).

Find the list of supported platforms for Percona Backup for MongoDB on the [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mongodb) page.

## System requirements

* At least 1GB RAM is required on every node for `pbm-agents` to operate successfully.
* All `pbm-agents` in the cluster must be able to connect to all config server replica set nodes that could become a new primary. In non-sharded replica set deployments, this means to connect to all the nodes that could become a new primary node. To become a primary, a node must meet the following criteria:

    * have `priority` greater than `0` and must be able to vote (`votes`: 1)
    * is not an arbiter (`arbiterOnly: false`)
    * is not hidden (`hidden: false`)
    * is not delayed 

* All `pbm-agents` in your deployment must be able to connect to the same [remote backup storage](details/storage-configuration.md) using the same credentials.

Note that networking issues like connection to the remote backup storage can also affect PBM performance. 

## Tutorials

You can use any of the easy-install guides but **we recommend using the package manager of your operating system** for a convenient and quick way to try the software first.

=== ":simple-windowsterminal: Package manager"
    
    Use the package manager of your operating system to install Percona Backup for MongoDB:

    * `apt` - for Debian and Ubuntu Linux
    * `yum` - for Red Hat Enterprise Linux and compatible Linux derivatives

    [Install from repositories :material-arrow-right:](install/repos.md){.md-button}

=== ":simple-docker: Docker"

     Get our Docker image and spin up PBM for a quick evaluation. 

     Check the Docker guide for step-by-step guidelines.

     [Run in Docker :material-arrow-right:](install/docker.md){.md-button}

=== ":simple-kubernetes: Kubernetes"

    **Percona Operator for Kubernetes** is a controller introduced to simplify complex deployments that require meticulous and secure database expertise. 

    Check the Quickstart guides how to deploy and run PBM on Kubernetes with Percona Operator for MongoDB.

    [Deploy in Kubernetes Quickstart :material-arrow-right:](https://docs.percona.com/percona-operator-for-mongodb/quickstart.html){.md-button}

=== ":octicons-file-code-16: Build from source"

    Have a full control over the installation by building PBM from source code.

    Check the guide below for step-by-step instructions.

    [Build from source :material-arrow-right:](install/source.md){.md-button}

=== ":octicons-download-16: Manual download"

    If you need to install PBM offline or prefer a specific version of it, check out the link below for a step-by-step guide and get access to the downloads directory.

    [Install from tarballs :material-arrow-right:](install/tarball.md){.md-button}


## Next steps

[Initial setup :material-arrow-right:](install/initial-setup.md){.md-button}



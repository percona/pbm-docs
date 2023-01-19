# Percona Backup for MongoDB Documentation


Percona Backup for MongoDB is an open-source, distributed and low-impact solution for consistent backups of [MongoDB sharded clusters and replica sets](deployments.md). It enables you to make logical, physical, incremental and selective backups and restores. [Point-in-Time recovery](usage/point-in-time-recovery.md) functionality allows you to recover your database to a specific timestamp. 

With Percona Backup for MongoDB you can  design and implement the effective backup strategy that considers the size and usage patterns of your database, the resources it utilizes and the goals of your organization. 

!!! note ""

    This is the documentation for the latest release, **PBM {{release}}** ([Release Notes](release-notes/{{release}}.md)).

## Why Percona Backup for MongoDB?

* Enterprise-grade features for free: 

    * [Logical backups and restores](details/logical.md)
    * [Physical (a.k.a. ‘hot’) backups and restores](details/physical.md). Available with Percona Server for MongoDB 4.2.15-16, 4.4.6-8, 5.0.2-1 and higher
    * [Point-in-time recovery](usage/point-in-time-recovery.md)
    * [Manual point-in-time recovery for any type of backup](usage/oplog-replay.md)
    * [Selective logical backups and restores](usage/selective-backup.md) (the [technical preview feature](reference/glossary.md#technical-preview-feature))
    * [Incremental physical backups](usage/incremental-backup.md) (the [technical preview feature](reference/glossary.md#technical-preview-feature)). Available with Percona Server for MongoDB.

* [Works for both sharded clusters and non-sharded replica sets](deployments.md)
* [Simple command-line management utility](reference/pbm-commands.md)
* Simple, [integrated-with-MongoDB authentication](initial-setup.md#external -authentication-support-in-percona-backup-for-mongodb)
* Distributed transaction consistency with MongoDB 4.2+
* Compatible with different storage types: [S3-compatible storage](details/storage-configuration.md#s3-compatible-storage), [Microsoft Azure Blob storage](details/storage-configuration.md#microsoft-azure-blob-storage), `filesystem` storage type for [locally-mounted remote filesystem backup servers](details/storage-configuration.md#remote-filesystem-server-storage)


The Percona Backup for MongoDB project is inherited from and replaces *mongodb_consistent_backup*, which is no longer actively developed or supported.

## Get started

* [Install Percona Backup for MongoDB](installation.md)
* [Set up Percona Backup for MongoDB](initial-setup.md)
* [Use Percona Backup for MongoDB](reference/pbm-commands.md) CLI to manage backups and restores

## Read more

* [How Percona Backup for MongoDB works](intro.md)
* [Percona Backup for MongoDB components](pbm-components.md)



[^1]: Tech Preview Features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.

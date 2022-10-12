# Percona Backup for MongoDB Documentation

Percona Backup for MongoDB is an open-source, distributed and low-impact solution for consistent backups of [MongoDB sharded clusters and replica sets](deployments.md).

As of version 1.7.0, Percona Backup for MongoDB supports both [physical and logical backups and restores](details/backup-types.md). [Point-in-Time recovery](usage/point-in-time-recovery.md) is currently supported only for logical backups.

!!! note

    Physical backups and restores is the technical preview feature [^1]. Before using them in production, we recommend that you test restoring from physical backups in your environment, and also use an alternative backup method for redundancy.

## Why Percona Backup for MongoDB?

* Enterprise-grade features for free: 

    * [Logical backups and restores](usage/start-backup.md)
    * [Physical (a.k.a. ‘hot’) backup and restore](details/backup-types.md). Available with Percona Server for MongoDB 4.2.15-16, 4.4.6-8, 5.0.2-1 and higher
    * [Point-in-time recovery](usage/point-in-time-recovery.md) (for logical backups only)
    * [Manual point-in-time recovery for any type of backup](usage/oplog-replay.md) (the technical preview feature [^1])
    * [Selective logical backups and restores](usage/selective-backup.md)

* [Works for both sharded clusters and non-sharded replica sets](deployments.md)
* [Simple command-line management utility](reference/pbm-commands.md)
* Simple, [integrated-with-MongoDB authentication](initial-setup.md#external -authentication-support-in-percona-backup-for-mongodb)
* Distributed transaction consistency with MongoDB 4.2+
* Can be used with any [S3-compatible storage](details/storage-configuration.md#s3-compatible-storage)
* Support for [Microsoft Azure Blob storage](details/storage-configuration.md#microsoft-azure-blob-storage)
* Supports `filesystem` storage type for [locally-mounted remote filesystem backup servers](details/storage-configuration.md#remote-filesystem-server-storage)


The Percona Backup for MongoDB project is inherited from and replaces *mongodb_consistent_backup*, which is no longer actively developed or supported.

## Get started

* [Install Percona Backup for MongoDB](installation.md)
* [Set up Percona Backup for MongoDB](initial-setup.md)
* [Use Percona Backup for MongoDB](reference/pbm-commands.md) CLI to manage backups and restores

## Read more

* [How Percona Backup for MongoDB works](intro.md)
* [Percona Backup for MongoDB components](pbm-components.md)



[^1]: Tech Preview Features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.

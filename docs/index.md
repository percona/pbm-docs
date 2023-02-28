# Percona Backup for MongoDB Documentation


Percona Backup for MongoDB (PBM) is an open source and distributed solution without notable performance and operating degradation for consistent backups and restore of [MongoDB sharded clusters and replica sets](deployments.md).

With Percona Backup for MongoDB, you can make backups on a running server and restore your database to a specific point in time using the command line. To do these tasks from a user interface, [use PBM with Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html).

!!! note ""

    This is the documentation for the latest release, **PBM {{release}}** ([Release Notes](release-notes/{{release}}.md)).

## What you can do with Percona Backup for MongoDB


![image](_images/backups-infographic.png)

* [Logical backups](details/logical.md) to back up and / or migrate data to different platforms and database versions
* [Physical backups](details/physical.md) to speed up performance for large (multi-terabyte) data sets
* [Selective backups](usage/selective-backup.md) to work with the desired data set
* [Incremental physical backups](usage/incremental-backup.md) to ensure that critical data is regularly backed up and to save on costs for storage and transfer
* [Restore the database in full](usage/restore.md) from a backup
* [Restore the database to a specific point in time](usage/point-in-time-recovery.md)
* [Restore only a selection of a database](usage/selective-backup.md#selective-restore) from a full backup
* [Replay oplog on top of EBS-snapshots](usage/oplog-replay.md)

[Explore features](details/backup-types.md){ .md-button .md-button }

## What's in it for you?

* Data consistency across clusters and replica sets
* No vendor lock-in due to a variety of [supported storage types](details/storage-configuration.md)
* Open source solution with [enterprise-grade features](comparison.md) 

[Install and get started](installation.md){ .md-button .md-button--primary }

## Go further with Percona Backup for MongoDB

* Learn [how Percona Backup for MongoDB works](intro.md)
* [Manage PBM](upgrade.md) 
* [Contribute to PBM](reference/contributing.md) 




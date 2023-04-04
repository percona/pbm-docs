# Percona Backup for MongoDB Documentation


Percona Backup for MongoDB (PBM) is an open source and distributed solution for consistent backups and restore of [MongoDB sharded clusters and replica sets](details/deployments.md). There is no notable performance nor operating degradation associated with PBM.

With Percona Backup for MongoDB, you can make backups on a running server and restore your database to a specific point in time using the command line. To do these tasks from a user interface, [use PBM with Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html).


!!! note ""

    This is the documentation for the latest release, **PBM {{release}}** ([Release Notes](release-notes/{{release}}.md)).

## What you can do

![image](_images/backups-infographic.png#only-light)
![image](_images/backups-infographic-dark.png#only-dark)

* [Logical backups](features/logical.md) to back up and / or migrate data to different platforms and database versions
* [Physical backups](features/physical.md) to speed up performance for large (multi-terabyte) data sets
* [Selective backups](features/selective-backup.md) to work with the desired data set
* [Incremental physical backups](features/incremental-backup.md) to ensure that critical data is regularly backed up and to save on costs for storage and transfer
* [Restore the full database or specific data set](usage/restore.md) from a backup
* [Restore the database to a specific point in time](features/point-in-time-recovery.md)
* [Replay oplog](usage/oplog-replay.md) on top of [EBS-snapshots](reference/glossary.md#ebs-snapshot)

[Explore features](features/backup-types.md){ .md-button .md-button }

## What's in it for you?

* Data consistency across clusters and replica sets
* A variety of [supported storage types](details/storage-configuration.md) means no vendor lock-in
* Open source solution with [enterprise-grade features](features/comparison.md) 

[Install and get started](installation.md){ .md-button .md-button }

## Go further with Percona Backup for MongoDB

* Learn [how Percona Backup for MongoDB works](intro.md)
* [Manage PBM](manage/upgrading.md) 
* [Contribute to PBM](reference/contributing.md) 




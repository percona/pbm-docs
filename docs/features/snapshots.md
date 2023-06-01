# API for snapshot-based physical backups

!!! admonition "Version added: [2.2.0](../release-notes/2.2.0.md)"

## Considerations 

1. This is a [technical preview feature](../reference/glossary.md#technical-preview-feature).
2. Supported only for full physical backups
3. Available only for Percona Backup for MongoDB running with Percona Server for MongoDB as PBM uses the [`$backupCursor and $backupCursorExtended aggregation stages`](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html). 

While a physical backup is a physical copy of your data directory, a snapshot is a point in time copy of your disk or a volume where the data files are stored. Restoring from snapshots is much faster and allows almost immediate access to data, while the database is unavailable during physical restore. Snapshot-based backups are especially useful for owners of large data sets with terabytes of data. Yet the snapshots don’t guarantee data consistency in sharded clusters.

This is where Percona Backup for MongoDB steps in. It provides the API to make snapshot-based physical backups and restores and ensures data consistency. As a result, database owners benefit from increased performance and reduced downtime, and are sure that their data remains consistent.

The snapshot-based physical backup / restore flow consists of three distinct stages:

* Preparing the database — done by PBM
* Copying files — done by the user / client app
* Completing the backup / restore — done by PBM. 

This is the first stage of the snapshot-based backups where you can make them manually using the API. Automated snapshot-based backups are planned for future releases.

## Make a backup

Before you start:

1. [Install](../installation.md) and [set up Percona Backup for MongoDB](../install/initial-setup.md)
2. Check that `pbm agent` is running with the [`pbm status`](../reference/pbm-commands.md#pbm-status) command

To make a snapshot-based backup, specify its type as external for [`pbm backup`](../reference/pbm-commands.md#pbm-backup) command:

```{.bash data-prompt="$"}
$ pbm backup -t external --list-files
```

PBM opens the `$backupCursor`, stores the backup metadata, prepares the database for file copy and prints the prompt similar to the following:

```{.text .no-copy}
Ready to copy data from:
<node-list>
```

You also see the backup name and the list of files to copy from each node. 

At this stage you can copy files to the storage / make a snapshot using the technology of your choice.

To check the backup progress, run the [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup) to check the backup state and what nodes are running backup.

After the file copy, run the following command to close the `$backupCursor` and complete the backup: 

```{.bash data-prompt="$"}
$ pbm backup-finish <backup_name>
```

## Restore a backup

To make a restore, run the following command:

```{.bash data-prompt="$"}
$ pbm restore [backup_name] --external 
```

Percona Backup for MongoDB prepares the database, provides the restore name and prompts you to copy the data:

```{.text .no-copy}
<example goes here>
``` 

Note that you must **copy the data on every node of the replica set / sharded cluster**. 

After you copied the files to the nodes, complete the restore with the following command:

```{.bash data-prompt="$"}
$ pbm restore-finish <restore_name> -c </path/to/pbm.conf.yaml>
```

At this stage PBM maintains data consistency and starts the cluster / replica set.


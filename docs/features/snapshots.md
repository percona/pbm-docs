# Snapshot-based physical backups

!!! admonition "Version added: [2.2.0](../release-notes/2.2.0.md)"

## Considerations 

1. Supported only for full backups
2. Available only if you run Percona Server for MongoDB in your environment as PBM uses the [`$backupCursor and $backupCursorExtended aggregation stages` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html). 

While a physical backup is a physical copy of your data directory, a snapshot is a point in time copy of your disk or a volume where the data files are stored. Restoring from snapshots is much faster and allows almost immediate access to data, however the database is unavailable during restore. Snapshot-based backups are especially useful for owners of large data sets with terabytes of data. Yet the snapshots don’t guarantee complete data consistency in sharded clusters.

This is where Percona Backup for MongoDB steps in. It provides the interface to make snapshot-based physical backups and restores and ensures data consistency. As a result, database owners benefit from increased performance and reduced downtime, and are sure that their data remains consistent.

The snapshot-based physical backup / restore flow consists of three distinct stages:

* Preparing the database — done by PBM
* Copying files — done by the user / client app
* Completing the backup / restore — done by PBM. 

This is the first stage of the snapshot-based backups where you can make them manually. Automated snapshot-based backups are planned for the future.

[Make a backup](../usage/backup-external.md){.md-button}
[Restore from a backup](../usage/restore-external.md){.md-button}



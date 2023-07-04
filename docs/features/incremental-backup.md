# Incremental physical backups

!!! admonition "Version added: [2.0.3](../release-notes/2.0.3.md)"

## Considerations

* :warning: Incremental backups made with Percona Backup for MongoDB prior to [2.1.0](../release-notes/2.1.0.md) are incompatible for restore with Percona Backup for MongoDB 2.1.0. This is because of the changed set of metadata files that are now stored in backups. These files are absent in backups made with previous PBM versions but are required for the restore with PBM 2.1.0.

    We recommend to make a new incremental base backup and start the incremental backup chain from it after the upgrade to Percona Backup for MongoDB 2.1.0 

* Incremental backup implementation is based on the [`$backupCursor`](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html) aggregation stage that is available only in Percona Server for MongoDB. Therefore, you must be running Percona Server for MongoDB in your deployment to use incremental physical backups.
* Incremental backups are supported for Percona Server for MongoDB starting with the following versions: [4.2.24-24](https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.24-24.html), [4.4.18](https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.18-18.html), [5.0.14-12](https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.14-12.html), [6.0.3-2](https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.3-2.html) and higher. 
* Due to [WiredTger restrictions in Log-Structured Merge (LSM) trees](https://source.wiredtiger.com/develop/backup.html#backup_incremental-block) behavior when the `$backupCursor` is opened, incremental backups are not available if the LSM tree is configured in the database.

Owners of large datasets may need to back up data frequently. Making full physical backups every time is costly in terms of storage space. Incremental physical backups come in handy in this scenario, enabling you to optimize backup strategy and reduce storage costs.

During incremental backups, Percona Backup for MongoDB saves only the data that was changed after the previous backup was taken. This results in faster backup / restore performance. Since incremental backups are smaller in size compared to full backups, you also save on costs for their storage and transfer in case of cloud deployments.

```mermaid
graph LR
  A[Full physical ] --> B([Increment 1 ]);
  B --> C([Increment 2 ]);
  C --> |.....| D([Increment n ]);
```

## Implementation specifics

Percona Backup for MongoDB tracks the backup history only on the node where the base incremental backup was taken. This means that subsequent incremental backups must always be run on that very node. To make this happen, Percona Backup for MongoDB tries to schedule backups on that same node.

If the node with the base incremental backup is down or unavailable, you must start the incremental backup chain anew on another node.

[Make a backup](../usage/start-backup.md){ .md-button .md-button }
[Restore a backup](../usage/restore.md){ .md-button .md-button }


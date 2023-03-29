# Incremental physical backups

!!! admonition "Version added: [2.0.3](../release-notes/2.0.3.md)"

## Considerations

* This is a [tech preview feature](../reference/glossary.md#technical-preview-feature). We recommend using it only for testing purposes. 

* Incremental backup implementation is based on the [`$backupCursor`](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html) aggregation stage that is available in only Percona Server for MongoDB. Therefore, you must be running Percona Server for MongoDB in your deployment to use incremental physical backups.

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


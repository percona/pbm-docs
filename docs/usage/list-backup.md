# List backups

!!! note

    As of version 1.4.0, the `pbm list` command provides the information only about completed backups. To check for running backups, use the [`pbm status`](../reference/pbm-commands.md#pbm-status). For more information, see [Percona Backup for MongoDB status](../manage/troubleshooting.md#percona-backup-for-mongodb-status).

    For Percona Backup for MongoDB version 1.3.4 and earlier,  the `pbm list` command provides the running backup listed with an
    ‘In progress’ label. When that is absent, the backup is complete.

Use the `pbm list` command to view all completed backups. 

```sh
pbm list
```

As of version 1.4.0, the **pbm list** output shows the time to which the sharded cluster / non-shared replica set will be returned to after the restore.

**Sample output**

```
Backup snapshots:
  2021-01-13T15:50:54Z [restore_to_time: 2021-01-13T15:53:40Z]
  2021-01-13T16:10:20Z [restore_to_time: 2021-01-13T16:13:00Z]
  2021-01-20T17:09:46Z [restore_to_time: 2021-01-20T17:10:33Z]
```

In logical backups, the completion time almost coincides with the backup finish time. To define the completion time, Percona Backup for MongoDB waits for the backup snapshot to finish on all cluster nodes. Then it captures the oplog from the backup start time up to that time.

In physical backups, the completion time is only a few seconds after the backup start time. By holding the `$backupCursor` open guarantees that the checkpoint data won’t change during the backup, and Percona Backup for MongoDB can define the completion time ahead.

The type of backups is available in the `pbm list` output starting with version 1.7.0.

```sh
pbm list

  Backup snapshots:
    2021-12-13T13:05:14Z <physical> [restore_to_time: 2021-12-13T13:05:17Z]
```

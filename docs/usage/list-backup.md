# List backups

!!! note

    As of version 1.4.0, the `pbm list` command provides the information only about completed backups. To check for running backups, use the [`pbm status`](../reference/pbm-commands.md#pbm-status). For more information, see [Percona Backup for MongoDB status](../manage/status.md).

    For Percona Backup for MongoDB version 1.3.4 and earlier, the `pbm list` command provides the running backup listed with an
    ‘In progress’ label. When that is absent, the backup is complete.

Use the `pbm list` command to view all completed backups. 

```{.bash data-prompt="$"}
$ pbm list
```

The output provides the following information:

* Backup name 
* Backup type: [logical](../features/logical.md), [physical](../features/physical.md), [selective](../features/selective-backup.md), [incremental](../features/incremental-backup.md). Available starting with version 1.7.0 
* The time to which the sharded cluster / non-shared replica set will be returned to after the restore. Available starting with version 1.4.0.
* If [point-in-time recovery](../features/point-in-time-recovery.md) is enabled, its status and the valid time ranges for the restore

**Sample output**

```{.text .no-copy}
Backup snapshots:
  2023-03-10T10:44:52Z <logical> [restore_to_time: 2023-03-10T10:44:56Z]
  2023-03-10T10:49:20Z <physical> [restore_to_time: 2023-03-10T10:49:23Z]
  2023-03-10T10:50:22Z <incremental> [restore_to_time: 2023-03-10T10:50:25Z]
  2023-03-10T10:51:02Z <incremental> [restore_to_time: 2023-03-10T10:51:04Z]
  2023-03-10T10:57:47Z <incremental> [restore_to_time: 2023-03-10T10:57:49Z]
  2023-03-10T11:04:25Z <incremental> [restore_to_time: 2023-03-10T11:04:27Z]
  2023-03-10T11:05:03Z <logical, selective> [restore_to_time: 2023-03-10T11:05:07Z]
```

## Restore to time

In logical backups, the completion time almost coincides with the backup finish time. To define the completion time, Percona Backup for MongoDB waits for the backup snapshot to finish on all cluster nodes. Then it captures the oplog from the backup start time up to that time.

In physical backups, the completion time is only a few seconds after the backup start time. By holding the `$backupCursor` open guarantees that the checkpoint data won’t change during the backup, and Percona Backup for MongoDB can define the completion time ahead.


## Useful links

[View detailed information about a backup](describe-backup.md)
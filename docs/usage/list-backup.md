# List backups

Use the `pbm list` command to view all completed backups. 

```bash
pbm list
```

The output provides the following information:

* Backup name 
* Backup type: [logical](../features/logical.md), [physical](../features/physical.md), [selective](../features/selective-backup.md), [incremental](../features/incremental-backup.md). Available starting with version 1.7.0 
* The time to which the sharded cluster / non-shared replica set will be returned to after the restore. Available starting with version 1.4.0.
* The duration of the completed backup
* If [point-in-time recovery](../features/point-in-time-recovery.md) is enabled, its status and the valid time ranges for the restore

??? example "Sample output"

    ```{.text .no-copy}
    Backup snapshots:
      NAME                      TYPE          PROFILE               SELECTIVE   BASE    RESTORE TIME         DURATION
      ---------------------------------------------------------------------------------------------------------------
      2026-08-24T12:17:24Z      physical                            no          no      2026-08-24T12:17:27  20s
      2026-08-24T12:17:58Z      logical                             no          no      2026-08-24T12:21:36  3m51s
    ```

## Restore to time

In logical backups, the completion time almost coincides with the backup finish time. To define the completion time, Percona Backup for MongoDB waits for the backup snapshot to finish on all cluster nodes. Then it captures the oplog from the backup start time up to that time.

In physical backups, the completion time is only a few seconds after the backup start time. By holding the `$backupCursor` open guarantees that the checkpoint data won't change during the backup, and Percona Backup for MongoDB can define the completion time ahead.


## Useful links

* [View detailed information about a backup](describe-backup.md)
* [Restore to a point-in-time](pitr-physical.md)
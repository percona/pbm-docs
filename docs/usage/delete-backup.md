# Delete backups

Use the ``pbm delete-backup`` command to delete either a specified backup snapshot or all backup snapshots older than the specified time.

!!! note 

    You can only delete a backup that is not running (has the “done” or the “error” state). To check the backup state, run the [`pbm status`](../reference/pbm-commands.md#pbm-status) command.

Starting with version 1.6.0, the command deletes only backup snapshots. Starting with version 2.0.0, you can also delete [selective backups](selective-backups.md). 

To [delete point-in-time recovery oplog slices](point-in-time-recovery.md#delete-oplog-slices), use the [`pbm delete-pitr`](../reference/pbm-commands.md#pbm-delete-pitr) command.


## Considerations

To ensure oplog continuity for [point-in-time restore](point-in-time-recovery.md#restore-to-the-point-in-time), the `pbm delete-backup` command deletes any backup(s) but for the following ones:

* A backup snapshot that can serve as the base for any point in time recovery and has point-in-time recovery time ranges deriving from it. To delete such a backup, first [delete the oplog slices](point-in-time-recovery.md#delete-oplog-slices) that are created  after the `restore-to time` value for this backup.

* The most recent backup if point-in-time recovery is enabled and there are no oplog slices following this backup yet.

To illustrate this, let’s take the following `pbm list` output:

```
Backup snapshots:
  2022-10-05T14:13:50Z <logical> [restore_to_time: 2022-10-05T14:13:55Z]
  2022-10-06T14:52:42Z <logical> [restore_to_time: 2022-10-06T14:52:47Z]
  2022-10-07T14:57:17Z <logical> [restore_to_time: 2022-10-07T14:57:22Z]

PITR <on>:
  2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z
```

You can delete a backup `2022-10-06T14:52:42Z` since it has no point-in-time oplog slices. You cannot delete the following backups:

- `2022-10-05T14:13:50Z` because it is the base for recovery to any point in time from the PITR time range `2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z`
- `2022-10-07T14:57:17Z` because PITR is enabled and there are no oplog slices following it yet.


## Behavior

To delete a backup, specify the `<backup_name>` as an argument.

```
pbm delete-backup 2021-12-20T13:45:59Z
```

By default, the ``pbm delete-backup`` command asks for your confirmation
to proceed with the deletion. To bypass it, add the `-f` or
`--force` flag.

```
pbm delete-backup --force 2021-04-20T13:45:59Z
```

To delete backups that were created before the specified time, pass the `--older-than` flag to the `pbm delete-backup` command. Specify the timestamp as an argument for `pbm delete-backup` in the following format:

* `%Y-%M-%DT%H:%M:%S` (for example, 2021-04-20T13:13:20Z) or
* `%Y-%M-%D` (2021-04-20).

### Example

View backups:

```sh
pbm list
```

**Output**:

```
Backup snapshots:
  2021-04-20T20:55:42Z
  2021-04-20T23:47:34Z
  2021-04-20T23:53:20Z
  2021-04-21T02:16:33Z
```

Delete backups created before the specified timestamp

```sh
pbm delete-backup -f --older-than 2021-04-21
```

**Output**:

```
Backup snapshots:
  2021-04-21T02:16:33Z
```



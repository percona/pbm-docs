# Point-in-time restore from logical backups

--8<-- "pitr-preparation.md"

## Procedure

Run [`pbm restore`](../reference/pbm-commands.md#pbm-restore) and specify the timestamp from the valid range:    

```{.bash data-prompt="$"}
$ pbm restore --time="2022-12-14T14:27:04"
```    

The timestamp you specify for the restore must be within the time ranges in the PITR section of `pbm list` output. Percona Backup for MongoDB automatically selects the most recent backup among logical, physical and incremental in relation to the specified timestamp and uses that as the base for the restore.    

To illustrate this behavior, let’s use the following `pbm list` output as the example.     

```{.bash .no-copy}
$ pbm list    

  2025-03-04T13:00:58Z [restore_to_time: 2025-03-04T13:01:23]
  2025-03-05T13:00:47Z [restore_to_time: 2025-03-05T13:01:11]
  2025-03-06T08:02:44Z [restore_to_time: 2025-03-06T08:03:09]
  2025-03-06T08:03:43Z [restore_to_time: 2025-03-06T08:04:08]
  2025-03-06T08:18:17Z [restore_to_time: 2025-03-06T08:18:41] 

PITR <off>:
  2025-03-04T13:01:24 - 2025-03-05T13:00:11
  2025-03-06T08:03:10 - 2025-03-06T08:18:29
  2025-03-06T08:18:42 - 2025-03-06T08:33:09
```    

For timestamp `2025-03-06T08:10:10`, the backup snapshot `2025-03-06T08:02:44Z [restore_to_time: 2025-03-06T08:03:09]` is used as the base for the restore as it is the most recent one.    

If you [select a backup snapshot for the restore with the `–-base-snapshot` option](#select-a-backup-snapshot-for-the-restore), the timestamp for the restore must also be later than the selected backup.    

!!! admonition "See also"    

    [Restore a backup](restore.md)    

### Post-restore steps    

A restore operation changes the time line of oplog events. Therefore, all oplog slices made after the restore time stamp and before the last backup become invalid. After the restore is complete, do the following:    

1. Make a new backup to serve as the starting point for oplog updates:    

    ```{.bash data-prompt="$"}
    $ pbm backup
    ```    

2. Re-enable point-in-time recovery to resume saving oplog slices:    

    ```{.bash data-prompt="$"}
    $ pbm config --set pitr.enabled=true
    ```

## Select a backup snapshot for the restore

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

You can recover your database to the specific point in time using any backup snapshot, and not only the most recent one. Run the `pbm restore` command with the `--base-snapshot=<backup_name>` flag where you specify the desired backup snapshot.

To restore from any backup snapshot, Percona Backup for MongoDB requires continuous oplog. After the backup snapshot is made and point-in-time recovery is re-enabled, it copies the oplog saved with the backup snapshot and creates oplog slices from the end time of the latest slice to the new starting point thus making the oplog continuous.

## Useful links

* [Restore a backup](restore.md)
* [Replay oplog from arbitrary start time](oplog-replay.md)



# Make a point-in-time restore

## Preconditions

Run [`pbm status`](../reference/pbm-commands.md#pbm-status) or [`pbm list`](../reference/pbm-commands.md#pbm-list) commands to check that the full backup snapshot exists and there are oplog slices.

## Before you start

1. Disable point-in-time recovery. A restore and point-in-time recovery oplog slicing are incompatible operations and cannot be run simultaneously. 

    ```sh
    pbm config --set pitr.enabled=false
    ```

2. Stop the balancer and `mongos` nodes
3. Make sure no writes are made to the database during restore. 

## From logical backups 

Run [`pbm restore`](../reference/pbm-commands.md#pbm-restore) and specify the timestamp from the valid range:

```sh
pbm restore --time="2022-12-14T14:27:04"
```

The timestamp you specify for the restore must be within the time ranges in the PITR section of `pbm list` output. Percona Backup for MongoDB automatically selects the most recent backup in relation to the specified timestamp and uses that as the base for the restore.

To illustrate this behavior, let’s use the following `pbm list` output as the example. 

```{.bash .no-copy}
pbm list

  2021-08-04T13:00:58Z [restore_to_time: 2021-08-04T13:01:23Z]
  2021-08-05T13:00:47Z [restore_to_time: 2021-08-05T13:01:11Z]
  2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09Z]
  2021-08-06T08:03:43Z [restore_to_time: 2021-08-06T08:04:08Z]
  2021-08-06T08:18:17Z [restore_to_time: 2021-08-06T08:18:41Z]

PITR <off>:
  2021-08-04T13:01:24 - 2021-08-05T13:00:11
  2021-08-06T08:03:10 - 2021-08-06T08:18:29
  2021-08-06T08:18:42 - 2021-08-06T08:33:09
```

For timestamp `2021-08-06T08:10:10`, the backup snapshot `2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09]` is used as the base for the restore as it is the most recent one.

If you [select a backup snapshot for the restore with the `–base-snapshot` option](#selecting-a-backup-snapshot-for-the-restore), the timestamp for the restore must also be later than the selected backup.

!!! admonition "See also"

    [Restore a backup](restore.md)

### Post-restore steps

A restore operation changes the time line of oplog events. Therefore, all oplog slices made after the restore time stamp and before the last backup become invalid. After the restore is complete, do the following:

1. Make a new backup to serve as the starting point for oplog updates:

    ```sh
    pbm backup
    ```

2. Re-enable point-in-time recovery to resume saving oplog slices:

    ```sh
    pbm config --set pitr.enabled=true
    ```

### Select a backup snapshot for the restore

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

You can recover your database to the specific point in time using any backup snapshot, and not only the most recent one. Run the `pbm restore` command with the `--base-snapshot=<backup_name>` flag where you specify the desired backup snapshot.

To restore from any backup snapshot, Percona Backup for MongoDB requires continuous oplog. After the backup snapshot is made and point-in-time recovery is re-enabled, it copies the oplog saved with the backup snapshot and creates oplog slices from the end time of the latest slice to the new starting point thus making the oplog continuous.

## Restore selected databases and collections

Before you start, read [known limitations for selective backups and restores](../usage/selective-backup.md#known-limitations-of-selective-backups-and-restores)

To restore the desired database or a collection to a point in time, run the ``pbm restore`` command as follows:

```sh
pbm restore --base-snapshot <backup_name> --time <timestamp> \
--ns <db.collection>
```

You can specify the selective backup as the base snapshot for the Point-in-time restore. In this case, Percona Backup for MongoDB restores only the namespace(s) included in this backup to the specified time.

Alternatively, you can use a full backup snapshot and restore the desired namespaces (databases or collections) up to the specific time from it. Specify them as the comma-separated list for the `pbm restore` command.

When point-in-time recovery is started, Percona Backup for MongoDB uses the provided base snapshot, restores the specified namespace(s) and replays oplog on top of it up to the specified time. If no base snapshot is provided, Percona Backup for MongoDB uses the most recent full backup snapshot.

## From physical backups

A point-in-time restore from a physical backup consists of two steps: 

1. Restore from the physical backup snapshot 
2. Manual replay of oplog events on top of it up to a specific timestamp.

!!! tip

    If you make only physical backup snapshots, set the `pitr.oplogOnly=true` configuration parameter to start oplog slicing without the mandatory logical base snapshot. 


To restore the backup snapshot, run the `pbm restore` command specifying the backup name:

```sh
pbm restore <backup_name>
```

To replay the oplog to the particular timestamp, do the following:
 
 1. Stop oplog slicing, if enabled, to release the lock.

 2. Run `pbm status` or `pbm list` commands to find oplog chunks available for replay.

 3. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags with the timestamps. Specify the `--start` timestamp at least 1 sec before the `restore_to_time` value for the backup snapshot that you restored.

     ```sh
     pbm oplog-replay --start="<timestamp>" --end="<timestamp>"
     ```

 4. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

!!! admonition "See also"

    * [Restore a backup](restore.md)
    * [Replay oplog from arbitrary start time](oplog-replay.md)



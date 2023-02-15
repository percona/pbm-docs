# Point-in-Time Recovery

!!! admonition "Version added: 1.3.0"

Point-in-time recovery is restoring a database up to a specific timestamp. Point-in-time recovery includes restoring the data from a backup snapshot and replaying all events that occurred to this data up to a specified time from [oplog slices](../reference/glossary.md#oplog-slice). Point-in-time recovery helps you prevent data loss during a disaster such as crashed database, accidental data deletion or drop of tables, unwanted update of multiple fields instead of a single one.

To enable point-in-time recovery, set the `pitr.enabled` configuration option.

=== "Command line"

     ```sh
     pbm config --set pitr.enabled=true
     ```

=== "Configuration file"

     ```yaml
     pitr:
       enabled: true
     ```

## Oplog slicing

When point-in-time recovery is enabled, the `pbm-agent` periodically saves consecutive slices of the [oplog](../reference/glossary.md#oplog). A method similar to the way replica set nodes elect a new primary is used to select the `pbm-agent` that saves the oplog slices. (Find more information in [pbm-agent](../details/architecture.md#pbm-agent).)

To start saving oplog, Percona Backup for MongoDB requires a backup snapshot. Therefore, make sure that a backup exists when you enable point-in-time recovery.

!!! important

    If you [reshard](https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/) a collection in MongoDB 5.0 and higher versions, make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

### Oplog duration

!!! admonition "Version added: 1.6.0"

By default, a slice covers a 10 minute span of oplog events. It can be shorter if point-in-time recovery is disabled or interrupted by the start of a backup snapshot operation.

You can change the duration of an oplog span via the configuration file. Specify the new value (in minutes) for the `pitr.oplogSpanMin` option.

=== "Command line"

     ```sh
     pbm config --set pitr.oplogSpanMin=5
     ```

=== "Configuration file"

     ```yaml
     pitr:
       oplogSpanMin: 5
     ```

If you set the new duration when the `pbm-agent` is making an oplog slice, the slice’s span is updated right away.

If the new duration is shorter, this triggers the `pbm-agent` to make a new slice with the updated span immediately. If the new duration is larger,  the `pbm-agent` makes the next slice with the updated span in its scheduled time.

### Compressed oplog slices 

!!! admonition "Version added: 1.7.0"

The oplog slices are saved with the `s2` compression method by default. You can specify a different compression method via the configuration file. Specify the new value for the `pitr.compression` option.

=== "Command line"

     ```sh
     pbm config --set pitr.compression=gzip
     ```

=== "Configuration file"

     ```yaml
     pitr:
       compression: gzip
     ```

Supported compression methods are: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`.

Additionally, you can override the compression level used by the compression method by setting the [`pitr.compressionLevel`](../reference/ptr-options.ms#pitrcompressionlevel) option. The default values differ for each compression level. 

Note that the higher value you specify, the more time and computing resources it will take to compress the data.

!!! note 

    You can use different compression methods for backup snapshots and point-in-time recovery slices. However, backup snapshot-related oplog is compressed with the same compression method as the backup itself.

The oplog slices are stored in the `pbmPitr` subdirectory in the [remote storage defined in the config](../details/storage-configuration.md#storage-config). A slice name reflects the start and end time this slice covers.

The [`pbm list`](../reference/pbm-commands.md#pbm-list) output includes the following information:

* Backup snapshots. As of version 1.4.0, it also shows the completion time
* Valid time ranges for recovery
* Point-in-time recovery status.

   ```sh
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

!!! note 

    If you just enabled point-in-time recovery, the time range list in the `pbm list` output is empty. It requires 10 minutes for the first chunk to appear in the list.

## Restore to the point in time

A restore and point-in-time recovery oplog slicing are incompatible operations and cannot be run simultaneously. You must disable point-in-time recovery before restoring a database:

```sh
pbm config --set pitr.enabled=false
```

### From logical backups 

Run [`pbm restore`](../reference/pbm-commands.md#pbm-restore) and specify the timestamp from the valid range:

```sh
pbm restore --time="2020-12-14T14:27:04"
```

Restoring to the point in time requires both a backup snapshot and oplog slices that can be replayed on top of this backup. The timestamp you specify for the restore must  be within the time ranges in the PITR section of `pbm list` output. Percona Backup for MongoDB automatically selects the most recent backup in relation to the specified timestamp and uses that as the base for the restore.

To illustrate this behavior, let’s use the `pbm list` output from the previous example. For timestamp `2021-08-06T08:10:10`, the backup snapshot `2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09]` is used as the base for the restore as it is the most recent one.

If you [select a backup snapshot for the restore with the `–base-snapshot` option](#selecting-a-backup-snapshot-for-the-restore), the timestamp for the restore must also be later than the selected backup.

!!! admonition "See also"

    [Restoring a backup](restore.md)

A restore operation changes the time line of oplog events. Therefore, all oplog slices made after the restore time stamp and before the last backup become invalid. After the restore is complete, make a new backup to serve as the starting point for oplog updates:

```sh
pbm backup
```

Re-enable point-in-time recovery to resume saving oplog slices:

```sh
pbm config --set pitr.enabled=true
```

#### Selecting a backup snapshot for the restore

!!! admonition "Version added: 1.6.0"

You can recover your database to the specific point in time using any backup snapshot, and not only the most recent one. Run the `pbm restore` command with the `--base-snapshot=<backup_name>` flag where you specify the desired backup snapshot.

To restore from any backup snapshot, Percona Backup for MongoDB requires continuous oplog. After the backup snapshot is made and point-in-time recovery is re-enabled, it copies the oplog saved with the backup snapshot and creates oplog slices from the end time of the latest slice to the new starting point thus making the oplog continuous.

### From physical backups

Making a point-in-time restore from a physical backup consists of two steps: firstly, restore the backup snapshot and then replay oplog events on top of it up to a specific timestamp.

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

    [Replay oplog from arbitrary start time](oplog-replay.md)

## Delete backups

!!! admonition "Version added: 1.6.0"

Backup snapshots and oplog slices are deleted using separate commands: [`pbm delete-backup`](../reference/pbm-commands.md#pbm-delete-backup) and [`pbm delete-pitr`](../reference/pbm-commands.md#pbm-delete-pitr) respectively.

### Delete backup snapshots

For Percona Backup for MongoDB 1.6.0 and later versions, see [delete backups](delete-backup.md#considerations) for backup deletion flow.

!!! admonition ""

    For Percona Backup for MongoDB 1.5.0 and earlier versions, when you delete a backup, all oplog slices that relate to this backup are deleted too. For example, you delete a backup snapshot `2020-07-24T18:13:09` while there is another snapshot `2020-08-05T04:27:55` created after it.  The **pbm-agent** deletes only oplog slices that relate to `2020-07-24T18:13:09`.

    The same applies if you delete backups older than the specified time.

    Note that when point-in-time recovery is enabled, the most recent backup snapshot and oplog slices that relate to it are not deleted.

### Delete oplog slices



Running `pbm delete-pitr` allows you to delete old and/or unnecessary slices and save storage space. To see the oplog slices, use the `pbm list` command.

If you have deleted the snapshot and want to delete the oplog slices, use the `pbm list --unbacked` command to view them.

You can either delete all chunks by passing the  `--all` flag. Or you can delete all slices that are made earlier than the specified time by passing the `--older-than` flag. In this case, specify the timestamp as an argument for `pbm delete-pitr` in the following format:

* `%Y-%M-%DT%H:%M:%S` (for example, 2021-07-20T10:01:18) or
* `%Y-%M-%D` (2021-07-20).

```sh
pbm delete-pitr --older-than 2021-07-20T10:01:18
```

To enable [point in time recovery](point-in-time-recovery.md#restore-to-the-point-in-time) from the most recent backup snapshot, Percona Backup for MongoDB does not delete slices that were made after that snapshot. For example, if the most recent snapshot is `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` and you specify the timestamp `2021-07-20T07:05:44`, Percona Backup for MongoDB deletes only slices that were made before `2021-07-20T07:05:23Z`.

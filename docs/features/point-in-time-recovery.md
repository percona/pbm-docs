# Point-in-time recovery

!!! admonition "Version added: [1.3.0](../release-notes/1.3.0.md)"

Point-in-time recovery is restoring a database up to a specific timestamp. This includes restoring the data from a backup snapshot and replaying all events that occurred to this data up to a specified time from [oplog slices](#oplog-slicing). 

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
| Helps you prevent data loss during a disaster such as a crashed database, accidental data deletion or drop of tables, and unwanted update of multiple fields instead of a single one. | Restore takes longer since it requires you to restore the backup and then replay oplog events on top of it.|

## Enable point-in-time recovery

Set the `pitr.enabled` configuration option to `true`.

=== "Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.enabled=true
     ```

=== "Configuration file"

     ```yaml
     pitr:
       enabled: true
     ```

The `pbm-agent` starts [saving consecutive slices of the oplog](#oplog-slicing) periodically. A method similar to the way replica set nodes elect a new primary is used to select the `pbm-agent` that saves the oplog slices. (Find more information in [pbm-agent](../details/pbm-agent.md).)


[Restore to a point-in-time](../usage/pitr-tutorial.md){ .md-button .md-button }

## Oplog slicing

To start saving [oplog slices](../reference/glossary.md#oplog), the following preconditions must be met:

=== "Logical backups"

    * A full logical backup snapshot is required. Make sure that a [backup exists](../usage/list-backup.md). See the [Make a backup](../usage/start-backup.md) guide to make a backup snapshot.
    * Point-in-time recovery routine is [enabled](#enable-point-in-time-recovery). 

=== "Physical backups"

    Enable point-in-time recovery routine and configure it to save oplog slices without the base backup.

    ```yaml
    pitr:
       enabled: true
       oplogOnly: true
    ```
    

If you just enabled point-in-time recovery, it requires 10 minutes for the first chunk to appear in the [`pbm list`](../reference/pbm-commands.md#pbm-list) output.

!!! important

    **For in MongoDB 5.0 and higher versions**

    If you [reshard](https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/) a collection, make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

### Oplog duration

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

By default, a slice covers a 10-minute span of oplog events. It can be shorter if point-in-time recovery is disabled or interrupted by the start of a backup snapshot operation.

You can change the duration of an oplog span via the configuration file. Specify the new value (in minutes) for the `pitr.oplogSpanMin` option.

=== "Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.oplogSpanMin=5
     ```

=== "Configuration file"

     ```yaml
     pitr:
       oplogSpanMin: 5
     ```

If you set the new duration when the `pbm-agent` is making an oplog slice, the slice’s span is updated right away.

If the new duration is shorter, this triggers the `pbm-agent` to make a new slice with the updated span immediately. If the new duration is larger,  the `pbm-agent` makes the next slice with the updated span in its scheduled time.

### Compressed oplog slices 

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)"

The oplog slices are saved with the `s2` compression method by default. You can specify a different compression method via the configuration file. Specify the new value for the [`pitr.compression`](../reference/pitr-options.md#pitrcompression) option.

=== "Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.compression=gzip
     ```

=== "Configuration file"

     ```yaml
     pitr:
       compression: gzip
     ```

Supported compression methods are: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`.

Additionally, you can override the compression level used by the compression method by setting the [`pitr.compressionLevel`](../reference/pitr-options.md#pitrcompressionlevel) option. The default values differ for each compression level. 

Note that the higher value you specify, the more time and computing resources it will take to compress the data.

!!! note 

    You can use different compression methods for backup snapshots and point-in-time recovery slices. However, backup snapshot-related oplog is compressed with the same compression method as the backup itself.

### View oplog slices

The oplog slices are stored in the `pbmPitr` subdirectory in the [remote storage defined in the config](../details/storage-configuration.md#storage-config). A slice name reflects the start and end time this slice covers.

The [`pbm list`](../reference/pbm-commands.md#pbm-list) output includes the following information:

* Backup snapshots. As of version 1.4.0, it also shows the completion time (renamed to the `restore_to_time` in version 2.0.0)
* Valid time ranges for recovery
* Point-in-time recovery status

   ```{.bash data-prompt="$"}
   $ pbm list

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



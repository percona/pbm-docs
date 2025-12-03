# Replay oplog from arbitrary start time

You can replay the [oplog](../reference/glossary.md#oplog) for a specific period on top of any backup: logical, physical, storage level snapshot (like [EBS-snapshot](../reference/glossary.md#ebs-snapshot)). You can save oplog slices without the mandatory base backup snapshot. This behavior is controlled by the [`pitr.oplogOnly`](../reference/pitr-options.md) configuration parameter:

```yaml
pitr:
   oplogOnly: true
```

By replaying these oplog slices on top of the backup snapshot with the [`pbm oplog-replay`](../reference/pbm-commands.md#pbm-oplog-replay) command, you can manually restore sharded clusters and non-sharded replica sets to a specific point in time from a backup made by any tool and not only by Percona Backup for MongoDB. Plus, you reduce time, storage space, and administration efforts on making the redundant base backup snapshot.

!!! warning

    Use the oplog replay functionality with caution, only when you are sure about the starting time from which to replay oplog. The oplog replay does not guarantee data consistency when restoring from any backup. However, it is less error-prone for backups made with Percona Backup for MongoDB.

## Ways to specify time for oplog replay

PBM uses MongoDB's timestamp format for oplog replay, which provides operation-level resolution. Each oplog operation is identified by `(epoch, ordinal)`, where `epoch` is the Unix time in seconds and `ordinal` distinguishes multiple operations within the same second. The specified operation is always included in the replay.

You can define the oplog replay stop point in two ways:

1. **By ISO timestamp**:  
   Specify an end time as an ISO timestamp (for example, `2025-01-02T15:00:00`). Use this method when you want to include all operations that occurred until the specified second.

2. **By MongoDB timestamp tuple**:  
   Specify the stop point as `epoch,ordinal` (e.g., `1764576382,20`). PBM includes all operations up to that exact operation. Use this method when you need precise control over which specific operations within a second to include. 

## Oplog replay for physical backups

!!! note ""

    Starting with version 2.2.0, oplog replay on top of a physical backups made with Percona Backup for MongoDB is done automatically as part of [point-in-time recovery](pitr-physical.md). 

This section describes how to **manually** replay oplog on top of physical backups made with Percona Backup for MongoDB version 2.1.0 and earlier.

After you [restored a physical backup](restore.md), do the following:

1. Stop point-in-time recovery, if enabled, to release the lock.

2. Run `pbm status` or `pbm list` commands to find oplog chunks available for replay.

3. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags. See [how you can specify the time](#ways-to-specify-time-for-oplog-replay).

    === "Use timestamp"

        ```bash
        pbm oplog-replay --start="{{year}}-01-02T15:00:00" --end="{{year}}-01-03T15:00:00"
        ```
    
    === "Use `epoch,ordinal`"

        For a fine-grained precision which exactly operations within a second to include, specify the values for the `--start` and `--end` flags as `epoch,ordinal` tuples.

        ```bash
        pbm oplog-replay --end “1764576382,100”
        ``` 

4. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

## Oplog replay for storage level snapshots

When making a backup, Percona Backup for MongoDB stops the point-in-time recovery. This is done to maintain data consistency after the restore.

Storage-level snapshots are saved with point-in-time recovery enabled. Thus, after the database restore from such a backup, point-in-time recovery is automatically enabled and starts oplog slicing. These new oplog slices might conflict with the existing oplogs saved during the backup. To replay the oplog in such a case, do the following after the restore:


1. Disable point-in-time recovery.
2. Delete the oplog slices that might have been created.
3. Re-sync the data from the storage.
4. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags with the timestamps.

    ```bash
    pbm oplog-replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"
    ```

5. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

[Known limitations](../features/known-limitations.md#oplog-replay-from-arbitrary-start-time){.md-button}



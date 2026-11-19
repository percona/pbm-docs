> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Replay oplog from arbitrary start time

You can have PBM save oplog slices without the mandatory base backup snapshot. This behavior is controlled by the [`pitr.oplogOnly`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pitr-options.html) configuration parameter:

```
pitr:
   oplogOnly: true
```

You can then replay the [oplog](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#oplog) for a specific period on top of any backup: logical, physical, storage-level snapshot (like [EBS-snapshot](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#ebs-snapshot)).

By replaying these oplog slices on top of a backup snapshot with the [`pbm oplog-replay`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-oplog-replay) command, you can manually restore sharded clusters and non-sharded replica sets to a specific point in time from a backup made by any tool and not only by Percona Backup for MongoDB. Plus, you reduce time, storage space, and administration efforts on making the redundant base backup snapshot.

Warning

Use the oplog replay functionality with caution, only when you are sure about the starting time from which to replay oplog.

## Ways to specify time for oplog replay

PBM uses MongoDB’s timestamp format for oplog replay, which provides operation-level resolution. Each oplog operation is identified by `(epoch, ordinal)`, where `epoch` is the Unix time in seconds and `ordinal` distinguishes multiple operations within the same second. The specified operation is always included in the replay.

You can define the oplog replay stop point in two ways:

1. **By ISO timestamp**:\
   Specify an end time as an ISO timestamp (for example, `2025-01-02T15:00:00`). Use this method when you want to include all operations that occurred until the specified second.
1. **By MongoDB timestamp tuple**:\
   Specify the stop point as `epoch,ordinal` (e.g., `1764576382,20`). PBM includes all operations up to that exact operation. Use this method when you need precise control over which specific operations within a second to include.

## Oplog replay for physical backups

Starting with version 2.2.0, oplog replay on top of physical backups made with Percona Backup for MongoDB is done automatically as part of [point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/pitr-physical.html).

This section describes how to **manually** replay oplog on top of physical backups. This is useful for backups made with Percona Backup for MongoDB version 2.1.0 and earlier, where oplog apply was not automatically done by PBM.

After you [restore a physical backup](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-physical.html), do the following:

1. Stop point-in-time recovery, if enabled, to release the lock.

1. Run `pbm status` or `pbm list` commands to find oplog chunks available for replay.

1. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags. See [how you can specify the time](#ways-to-specify-time-for-oplog-replay).

   **Use timestamp**

   ```
   pbm oplog-replay --start="2026-01-02T15:00:00" --end="2026-01-03T15:00:00"
   ```

   **Use epoch,ordinal**

   For a fine-grained precision which exactly operations within a second to include, specify the values for the `--start` and `--end` flags as `epoch,ordinal` tuples.

   ```
   pbm oplog-replay --end “1764576382,100”
   ```

1. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

## Oplog replay for storage-level snapshots

This section describes how to **manually** replay oplog on top of snapshot backups.

After you [restore a snapshot backup](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-external.html), do the following:

1. Start all `mongod` nodes

1. Start all `pbm-agents`

1. Run the following command to resync the backup list with the storage:

   ```
   pbm config --force-resync
   ```

1. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags. Use the `last_write_time` from the output of `pbm describe-backup` as the start. See [how you can specify the time](#ways-to-specify-time-for-oplog-replay).

   ```
   pbm oplog-replay --start="2026-01-02T15:00:00" --end="2026-01-03T15:00:00"
   ```

1. Start the balancer and start `mongos` nodes.

1. Make a fresh backup to serve as the new base for future restores.

[Known limitations](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/known-limitations.html#oplog-replay-from-arbitrary-start-time)

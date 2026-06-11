# Make a physical backup

--8<-- "prepare-backup.md"

## Procedure

During a *physical* backup, Percona Backup for MongoDB  copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage. 

To start a backup, run the following command:

```bash
pbm backup --type=physical
```
     
!!! warning 

    During the period the backup cursor is open, database checkpoints can be created, but no checkpoints can be deleted. This may result in significant file growth.
    
Starting with [2.4.0](../release-notes/2.4.0.md), PBM doesn't stop [point-in-time recovery oplog slicing](../features/point-in-time-recovery.md#oplog-slicing), if it's enabled, but runs it in parallel. This ensures [point-in-time recovery](pitr-tutorial.md) to any timestamp if it takes too long (e.g. hours) to make a backup snapshot.

## Parallel file copy for filesystem storage

For physical backups stored on a filesystem, you can control how many files PBM processes in parallel during the backup operation. Increasing parallelism can improve performance by allowing multiple files to be copied simultaneously.

To copy files in parallel, set `backup.numParallelFiles` in the PBM configuration:

```yaml
backup:
  numParallelFiles: 4
```

Or pass `--num-parallel-files`:

```sh
pbm backup --type=physical --num-parallel-files=4
```

!!! note
    Parallel file copy applies to **physical backups and incremental physical backups stored on filesystem only**. It has no effect on logical backups or on any S3-compatible storage, regardless of backup type.

## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-physical.md){.md-button}
[Make a point-in-time recovery](pitr-physical.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make a physical backup

## Before you start

Before taking a backup, verify PBM is installed and running.

1. [Install](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [set up Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

1. Check that `pbm agent` is running with the [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) command

1. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them.

   To check the version, run the following commands:

   - [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of pbm-agents
   - [`pbm version`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of PBM CLI.

## Procedure

During a *physical* backup, Percona Backup for MongoDB copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage.

To start a backup, run the following command:

```
pbm backup --type=physical
```

Warning

During the period the backup cursor is open, database checkpoints can be created, but no checkpoints can be deleted. This may result in significant file growth.

Starting with [2.4.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.4.0.html), PBM doesn’t stop [point-in-time recovery oplog slicing](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/point-in-time-recovery.html#oplog-slicing), if it’s enabled, but runs it in parallel. This ensures [point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/pitr-tutorial.html) to any timestamp if it takes too long (e.g. hours) to make a backup snapshot.

## Parallel file copy for filesystem storage

For physical backups stored on a filesystem, you can control how many files PBM processes in parallel during the backup operation. Increasing parallelism can improve performance by allowing multiple files to be copied simultaneously.

To copy files in parallel, set `backup.numParallelFiles` in the PBM configuration:

```
backup:
  numParallelFiles: 4
```

Or pass `--num-parallel-files`:

```
pbm backup --type=physical --num-parallel-files=4
```

Note

Parallel file copy applies to **physical backups and incremental physical backups stored on filesystem only**. It has no effect on logical backups or on any S3-compatible storage, regardless of backup type.

## Next steps

[List backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/list-backup.html) [Make a restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-physical.html) [Make a point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/pitr-physical.html)

## Useful links

- [Backup and restore types](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)
- [Schedule backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/schedule-backup.html)

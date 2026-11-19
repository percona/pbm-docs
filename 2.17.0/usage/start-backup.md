> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make a logical backup

## Before you start

Before taking a backup, verify PBM is installed and running.

1. [Install](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [set up Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

1. Check that `pbm agent` is running with the [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) command

1. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them.

   To check the version, run the following commands:

   - [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of pbm-agents
   - [`pbm version`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of PBM CLI.

## Procedure

Warning

Sharded time series collections are not supported. If you use them in your deployment, you won’t be able to make a backup.

To make a backup, run the following command:

```
pbm backup --type=logical
```

Logical backup is the default one so you can bypass the `--type` flag.

During *logical* backups, Percona Backup for MongoDB copies the actual data to the backup storage.

Starting with version 2.0.0, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is now the default data format since it allows [selective restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-selective.html). Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB.

## Next steps

[List backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/list-backup.html) [Make a restore from a logical backup](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore.html) [Make a point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/pitr-tutorial.html)

## Useful links

- [Track backup progress](https://docs.percona.com/percona-backup-mongodb/2.17.0/troubleshoot/status.html#backup-progress-tracking)
- [Schedule backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/schedule-backup.html)

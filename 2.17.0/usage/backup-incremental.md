> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make incremental backups

## Before you start

Before taking a backup, verify PBM is installed and running.

1. [Install](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [set up Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

1. Check that `pbm agent` is running with the [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) command

1. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them.

   To check the version, run the following commands:

   - [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of pbm-agents
   - [`pbm version`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of PBM CLI.

## Procedure

1. To start incremental backups, first make a full incremental backup. It will serve as the base for subsequent incremental backups:

   ```
   pbm backup --type incremental --base
   ```

   The `pbm-agent` starts tracking the incremental backup history to be able to calculate and save the difference in data blocks.

1. Run regular incremental backups:

   ```
   pbm backup --type incremental
   ```

The incremental backup history looks like this:

Sample output

```
Snapshots:
    2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
    2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
    2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
    2022-11-24T14:45:53Z 220.13GB <incremental, base> [restore_to_time: 2022-11-24T14:45:55Z]
```

## Next steps

[List backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/list-backup.html) [Make a restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-incremental.html)

## Useful links

- [Backup and restore types](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)
- [Schedule backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/schedule-backup.html)

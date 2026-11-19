> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make an snapshot-based backup

## Before you start

Before taking a backup, verify PBM is installed and running.

1. [Install](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [set up Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

1. Check that `pbm agent` is running with the [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) command

1. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them.

   To check the version, run the following commands:

   - [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of pbm-agents
   - [`pbm version`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of PBM CLI.

## Procedure

1. To make a snapshot-based backup, run the [`pbm backup`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-backup) command with the type `external`:

   ```
   pbm backup -t external
   ```

   When executing the command, PBM does the following:

   - opens the `$backupCursor`
   - prepares the database for file copy
   - stores the backup metadata on the storage and adds it to the files to copy
   - prints the prompt similar to the following:

   ```
   Ready to copy data from:
   <node-list>
   ```

   In case of replica sets, `<node-list>` will include a single node. In case of sharded clusters, `<node-list>` will include a config node, and a node for each shard. Node [priority](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/backup-priority.html) affects the nodes that are selected and included in `<node-list>`.

   You also see the backup name.

1. (Optional) You can check the backup progress with the [`pbm describe-backup`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-describe-backup). The command output provides the backup state and what nodes are running backup.

1. At this stage, you need to copy the `dataDir` contents of each node in the `<node-list>` to the storage / make a snapshot using the technology of your choice.

1. After the copy/snapshot is complete, run the following command to close the `$backupCursor` and finish the backup:

   ```
   pbm backup-finish <backup_name>
   ```

## Next steps

[List backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/list-backup.html) [Make a restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-external.html)

## Useful links

- [Backup and restore types](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)
- [Schedule backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/schedule-backup.html)

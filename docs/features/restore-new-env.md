# Restore from a backup into a new environment

To restore a backup from one environment to another, ensure the following:

1. Percona Backup for MongoDB configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store. Once you run [`pbm list`](../reference/pbm-commands.md#pbm-list) and see the backups made from the original environment, then you can run the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command.

2. Don't run [`pbm backup`](../reference/pbm-commands.md#pbm-backup) from the new environment while Percona Backup for MongoDB configuration is pointing to the remote storage location of the original environment.

> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Restore from a backup into a new environment

To restore a backup from one environment to a new one, ensure the following:

1. Percona Backup for MongoDB configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store.
1. (Optional) Before running a point-in-time recovery, you can run the `pbm config --force-resync` command to synchronize the metadata between the original and new environments. This makes the new environment be aware of the most recent state of the backup storage.
1. Don’t run [`pbm backup`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-backup) from the new environment while Percona Backup for MongoDB configuration is pointing to the remote storage location of the original environment.
1. Don’t [enable point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/point-in-time-recovery.html) on the new environment while Percona Backup for MongoDB configuration is pointing to the remote storage location of the original environment.

Once you run [`pbm list`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-list) and see the backups made from the original environment, then you can run the [`pbm restore`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-restore) command.

After the restore is done, reconfigure PBM to point to the remote storage in the new environment to stop the original one from producing backup data.

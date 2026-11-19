> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Uninstall Percona Backup for MongoDB

Follow these steps to remove Percona Backup for MongoDB (PBM) from your environment.

## Procedure

1. Confirm no backups are running

   Check that no backup or restore is currently in progress:

   ```
   pbm list
   ```

   If an operation is still running, wait for it to finish (or [cancel it](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-cancel-backup)) before continuing. Removing PBM while an operation is active can leave your backup storage or control collections in an inconsistent state.

1. Disable point in time recovery

   If point in time recovery is still enabled, PBM will just start writing new oplog slices again right after cleanup

   ```
   pbm config --set pitr.enabled=false
   ```

1. (Optional) Delete backups from remote storage

   If you no longer need the backups PBM created, delete them from the remote storage using the `pbm cleanup` command. For example:

   ```
   pbm cleanup --older-than=0d --yes
   ```

   Warning

   If you’re using PBM’s multi-storage feature, you’ll need to repeat the cleanup with –profile for each storage profile — pbm cleanup only targets the currently active one by default.

1. Uninstall the pbm-agent and pbm executables

   Run these commands on every node where PBM is installed.

   **On Debian and Ubuntu**

   ```
   sudo apt remove percona-backup-mongodb
   ```

   **On RHEL and derivatives**

   ```
   sudo yum remove percona-backup-mongodb
   ```

   If you installed PBM a different way (tarball, source build, Docker), remove the binaries and any systemd service/config files manually instead. See [Install Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) for details on how PBM was set up on your system.

1. Drop the PBM control collections

   PBM stores its state in a set of collections in the `admin` database — on the config server replica set for a sharded cluster, or on the replica set itself otherwise. Connect with `mongosh` and drop them:

   ```
   use admin
   db.pbmAgents.drop()
   db.pbmBackups.drop()
   db.pbmCmd.drop()
   db.pbmConfig.drop()
   db.pbmLock.drop()
   db.pbmLockOp.drop()
   db.pbmLog.drop()
   db.pbmOpLog.drop()
   db.pbmPITR.drop()
   db.pbmPITRChunks.drop()
   db.pbmPITRState.drop()
   db.pbmRestores.drop()
   db.pbmStatus.drop()
   ```

1. Drop the PBM database user

   Drop the MongoDB user PBM used to authenticate (commonly named `pbm` or similar — check what you used when [setting up authentication](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/configure-authentication.html)):

   ```
   use admin
   db.dropUser("pbm")
   ```

   **In a sharded cluster**, run this same command on **each shard’s primary** as well as on the **config server replica set** — dropping the user in one place doesn’t remove it from the others.

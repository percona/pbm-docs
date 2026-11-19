> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Adjust PBM configuration when the number of replica set members changes

You may add or remove replica set members depending on your workload. With the default configuration for node priorities and `mongod` binary paths, PBM scales up or down automatically.

If you adjusted any of these configurations, here’s what you need to do to ensure that PBM continues to operate without errors:

## Add a new replica set member

1. [Install Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) on a new `mongod` node

1. Add a new `mongod` node to a replica set

1. [Set up and start](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html) `pbm-agent` on a new node.

   Before doing so, you may see the following error in `pbm status` output on your running replica set: `rs202:30202 [S]: pbm-agent [v2.17.0] FAILED status: ERROR with lost agent, last heartbeat: 1748531485`. This is expected behavior.

1. Adjust PBM configuration for [node priorities for backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/backup-priority.html), [oplog slices](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/point-in-time-recovery.html#adjust-node-priority-for-oplog-slices) or [define custom paths to mongod binaries](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-physical.html#define-a-mongod-binary-location) to include the new node.

## Remove a replica set member

1. Edit the PBM configuration for [node priorities for backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/backup-priority.html), [oplog slices](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/point-in-time-recovery.html#adjust-node-priority-for-oplog-slices) or [define custom paths to mongod binaries](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-physical.html#define-a-mongod-binary-location) to exclude the node you plan to remove.

1. Stop `pbm-agent` on the node you plan to remove:

   ```
   sudo systemctl stop pbm-agent
   ```

   You may see the error like the following in the `pbm status` output on the remaining nodes: `- rs202:30202 [S]: pbm-agent [v2.17.0] FAILED status: > ERROR with lost agent, last heartbeat: 1748531485`. This is expected behavior.

1. Remove the replica set member

1. `pbm status` should report the updated status without errors.

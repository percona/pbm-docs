# Adjust PBM configuration when the number of replica set members changes

You may add or remove replica set members depending on your workload. With the default configuration for node priorities and `mongod` binary paths, PBM scales up or down automatically. 

If you adjusted any of these configurations, here's what you need to do to ensure that PBM continues to operate without errors:

## Add a new replica set member {.power-number}

1. [Install Percona Backup for MongoDB](../installation.md) on a new `mongod` node
2. Add a new `mongod` node to a replica set 
3. [Set up and start](../install/initial-setup.md) `pbm-agent` on a new node. 

    Before doing so,  you may see the following error in `pbm status` output on your running replica set: `rs202:30202 [S]: pbm-agent [v{{release}}] FAILED status: ERROR with lost agent, last heartbeat: 1748531485`. This is expected behavior.

4. Adjust PBM configuration for [node priorities for backups](../usage/backup-priority.md), [oplog slices](../features/point-in-time-recovery.md#adjust-node-priority-for-oplog-slices) or [define custom paths to mongod binaries](../usage/restore-physical.md#define-a-mongod-binary-location) to include the new node.

## Remove a replica set member {.power-number}

1. Edit the PBM configuration for [node priorities for backups](../usage/backup-priority.md), [oplog slices](../features/point-in-time-recovery.md#adjust-node-priority-for-oplog-slices) or [define custom paths to mongod binaries](../usage/restore-physical.md#define-a-mongod-binary-location) to exclude the node you plan to remove.
2. Stop `pbm-agent` on the node you plan to remove:

    ```bash
    sudo systemctl stop pbm-agent
    ```

    You may see the error like the following in the `pbm status` output on the remaining nodes:
    `- rs202:30202 [S]: pbm-agent [v{{release}}] FAILED status: > ERROR with lost agent, last heartbeat: 1748531485`. This is expected behavior.

3. Remove the replica set member
4. `pbm status` should report the updated status without errors.



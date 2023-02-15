# Restore a backup

!!! warning 

    Backups made with Percona Backup for MongoDB prior to v1.5.0 are incompatible for restore with Percona Backup for MongoDB v1.5.0 and later. This is because processing of system collections `Users` and `Roles` has changed: in v1.5.0, `Users` and `Roles` are copied to temporary collection during backup and must be present in the backup during restore. In earlier versions of Percona Backup for MongoDB, `Users` and `Roles` are copied to a temporary collection during restore. Therefore, restoring from these backups with Percona Backup for MongoDB v1.5.0 isn’t possible.

    The recommended approach is to make a fresh backup after upgrading Percona Backup for MongoDB to version 1.5.0.

To restore a backup, use the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command supplying the backup name from which you intend to restore. Percona Backup for MongoDB identifies the type of the backup (physical, logical or [incremental](incremental-backup.md)) and restores the database up to the [backup completion time](../reference/glossary.md#completion-time) (available in `pbm list` output as of version 1.4.0).


## Logical restore

!!! important 

    **Restore considerations**

    1. Percona Backup for MongoDB is designed to be a full-database restore tool. As of version <=1.x, it performs a full all-databases, all collections restore and does not offer an option to restore only a subset of collections in the backup, as MongoDB’s `mongodump` tool does. But to avoid surprising `mongodump` users, as of versions 1.x, Percona Backup for MongoDB replicates `mongodump’s` behavior to only drop collections in the backup. It does not drop collections that are created new after the time of the backup and before the restore. Run a `db.dropDatabase()` manually in all non-system databases (these are all databases except “local”, “config” and “admin”) before running `pbm restore` if you want to guarantee that the post-restore database only includes collections that are in the backup.

    2. Whilst the restore is running, prevent clients from accessing the database. The data will naturally be incomplete whilst the restore is in progress, and writes the clients make cause the final restored data to differ from the backed-up data.

    3. If you enabled [Point-in-Time Recovery](point-in-time-recovery.md), disable it before running pbm restore. This is because Point-in-Time Recovery incremental backups and restore are incompatible operations and cannot be run together.

### Preconditions for the restore in sharded clusters

As preconditions for restoring from a backup in a sharded cluster, complete the following steps:

1. Stop the balancer.

2. Shut down all `mongos` nodes to stop clients from accessing the database while restore is in progress. This ensures that the final restored data doesn’t differ from the backed-up data.

3. Disable point-in-time recovery if it is enabled. To learn more about point-in-time recovery, see [Point-in-Time Recovery](point-in-time-recovery.md).

```sh
pbm restore 2019-06-09T07:03:50Z
```

Note that you can restore a sharded backup only into a sharded environment. It can be your existing cluster or a new one. To learn how to restore a backup into a new environment, see [Restoring a backup into a new environment](#restoring-a-backup-into-a-new-environment).

During the restore, the `pbm-agent` processes write data to primary nodes in the cluster. The following diagram shows the restore flow.

![image](../_images/pbm-restore-shard.png)

After a cluster’s restore is complete, restart all `mongos` nodes to reload the sharding metadata.

### Adjust memory consumption

Starting with version 1.3.2, Percona Backup for MongoDB config includes the restore options to adjust the memory consumption by the `pbm-agent` in environments with tight memory bounds. This allows preventing out of memory errors during the restore operation.

```yaml
restore:
  batchSize: 500
  numInsertionWorkers: 10
```

The default values were adjusted to fit the setups with the memory allocation of 1GB and less for the agent.

!!! note 

    The lower the values, the less memory is allocated for the restore. However, the performance decreases too.

## Physical restores

!!! important

    The MongoDB version for both backup and restore data must be within the same major release.

During the physical restore, `pbm-agent` processes stop the `mongod` nodes, clean up the data directory and copy the data from the storage onto every node.

The following diagram shows the physical restore flow:

![image](../_images/pbm-phys-restore-shard.png)

After the restore is complete, do the following:

1. Restart all `mongod` nodes

2. Restart all `pbm-agents`

3. Run the following command to resync the backup list with the storage:

 ```
 $ pbm config --force-resync
 ``` 

4. Start the balancer

### Tracking restore progress

You can [track physical restore progress](restore-progress.md) in Percona Backup for MongoDB version 2.0.0 and higher. 

For Percona Backup for MongoDB 1.8.1 and lower, the options to track the restore progress are:

* Check the stderr logs of the leader `pbm-agent`. The leader ID is printed once the restore has started.

* Check the status in the metadata file created on the remote storage for the restore. This file is in the root of the storage path and has the format `.pbm.restore/<restore_timestamp>.json`


### Physical restore known limitations

Physical restores are not supported for deployments with arbiter nodes. 

      

### Physical restores with data-at-rest encryption

!!! admonition "Version added: 2.0.0"

You can backup and restore the encrypted data at rest. Thereby you ensure data safety and can also comply with security requirements such as GDPR, HIPAA, PCI DSS, or PHI.

This is how it works: 

During a backup, Percona Backup for MongoDB stores the encryption settings in the backup metadata. This allows you to verify them using the `pbm describe-backup` command. Note that the encryption key is not stored nor shown.

!!! important

    Make sure that you know what encryption key was used and store it as this key is required for the restore.

To restore the encrypted data from the backup, do the following:

1. Put the encryption key / specify the path to the key on at least one node of every replica set. 

2. Configure data-at-rest encryption in your destination cluster or replica set. 

During the restore, Percona Backup for MongoDB restores the data on the node where the encryption key matches the one with which the backed up data was encrypted. The other nodes are not restored, so the restore has the "partially done" status. You can start this node and initiate the replica set. The remaining nodes receive the data as the result of the initial sync from the restored node. 

Alternatively, you can place the encryption key to all nodes of the replica set. Then the restore is successful and complete on all nodes. This approach is faster and may suit for large data sets (terabytes of data). However, we recommend to rotate the encryption keys afterwards. Note, that key rotation is not available after the restore [for data-at-rest encryption with HashiCorp Vault key server](https://docs.percona.com/percona-server-for-mongodb/latest/vault.html#vault). In this case, consider using the scenario with partially done restore. 

### Define `mongod` binary location

!!! admonition "Version added: 2.0.4"

During physical restores, Percona Backup for MongoDB performs several internal restarts of the database. For this, it uses the default path to the `mongod` binaries to access the database. If you have defined the custom path to the `mongod` binaries, you can make Percona Backup for MongoDB aware of it by specifying the path in the configuration file: 

```yaml
restore:
    mongodLocation: /path/to/mongod
    mongodLocationMap:
       "node01:27017": /path/to/mongod
       "node03:27017": /another/path/to/mongod
```

If, for some reason, you have different paths to `mongod` binaries on every node of your cluster / replica set, use the `mongodLocationMap` option to specify your custom paths for each node.

## Restoring a backup into a new environment

To restore a backup from one environment to another, consider the following key points about the destination environment:


* Replica set names (both the config servers and the shards) in your new destination cluster and in the cluster that was backed up must be exactly the same.


* Percona Backup for MongoDB configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store. Once you run **pbm list** and see the backups made from the original environment, then you can run the **pbm restore** command.

Of course, make sure not to run **pbm backup** from the new environment whilst the Percona Backup for MongoDB config is pointing to the remote storage location of the original environment.

## Restoring into a cluster / replica set with a different name

Starting with version 1.8.0, you can restore **logical backups** into a new environment that has the same or more number of shards and these shards have different replica set names.

To restore data to the environment with different replica set names, configure the name mapping between the source and target environments. You can either set the `PBM_REPLSET_REMAPPING` environment variable for `pbm` CLI or use the `--replset-remapping` flag for PBM commands. The mapping format is `<rsTarget>=<rsSource>`.

!!! important

    Configure replica set name mapping for all shards in your cluster. Otherwise, Percona Backup for MongoDB attempts to restore the unspecified shard to the target shard with the same name. If there is no shard with such name or it is already mapped to another source shard, the restore fails.

Configure the replica set name mapping:


=== "Using the environment variable for `pbm` CLI in your shell"

    ```
    $ export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"
    ``` 

=== "Using the command line"

    ```
    $ pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"
    ```

    The `--replset-remapping` flag is available for the following commands: `pbm restore`, `pbm list`, `pbm status`, `pbm oplog-replay`.

!!! note 

    Don’t forget to make a fresh backup on the new environment after the restore is complete.

This ability to restore data to clusters with different replica set names and the number of shards extends the set of environments compatible for the restore.


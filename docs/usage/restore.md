# Restore a backup

To restore a backup, use the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command supplying the backup name from which you intend to restore. Percona Backup for MongoDB identifies the type of the backup (physical, logical or [incremental](../features/incremental-backup.md)) and restores the database up to the [restore_to_time](../reference/glossary.md#completion-time) timestamp (available in `pbm list` output starting with version 1.4.0).

## Considerations

=== ":material-data-matrix: Logical"

    1. While the restore is running, prevent clients from accessing the database. The data will naturally be incomplete while the restore is in progress, and writes the clients make cause the final restored data to differ from the backed-up data.

    2. Backups made with Percona Backup for MongoDB prior to v1.5.0 are incompatible for restore with Percona Backup for MongoDB v1.5.0 and later. This is because processing of system collections `Users` and `Roles` has changed: in v1.5.0, `Users` and `Roles` are copied to temporary collection during backup and must be present in the backup during restore. In earlier versions of Percona Backup for MongoDB, `Users` and `Roles` are copied to a temporary collection during restore. Therefore, restoring from these backups with Percona Backup for MongoDB v1.5.0 isn’t possible.

        The recommended approach is to make a fresh backup after upgrading Percona Backup for MongoDB to version 1.5.0.

    3. For versions earlier than 1.x, Percona Backup for MongoDB performs a full all-databases, all collections restore and does not offer an option to restore only a subset of collections in the backup, as MongoDB’s `mongodump` tool does.

    4. Starting with versions 1.x, Percona Backup for MongoDB replicates `mongodump’s` behavior to only drop collections in the backup. It does not drop collections that are created new after the time of the backup and before the restore. Run a `db.dropDatabase()` manually in all non-system databases (these are all databases except “local”, “config” and “admin”) before running `pbm restore` if you want to guarantee that the post-restore database only includes collections that are in the backup.

=== ":material-database-refresh-outline: Physical"

    1. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
    2. Make sure all nodes in the cluster are healthy (i.e. either PRIMARY or SECONDARY). Each pbm-agent needs to be able to connect to its local node and run queries in order to perform the restore.
    3. For PBM versions before 2.1.0, physical restores are not supported for deployments with arbiter nodes.

=== ":simple-databricks: Incremental"

    1. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
    2. Incremental backups made with PBM before PBM 2.1.0 are incompatible for restore with PBM 2.1.0 and onwards.
    3. Physical restores are not supported for deployments with arbiter nodes.

=== ":material-database-export: Snapshot-based"

    1. Supported only for full physical backups
    2. Available only if you run Percona Server for MongoDB in your environment as PBM uses the [`$backupCursor and $backupCursorExtended aggregation stages` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html).

## Before you start

=== ":material-data-matrix: Logical"

    1. Stop the balancer and disable chunks autosplit. To verify that both are disabled, run the following command:

        ```{.javascript data-prompt=">"}
        > sh.status()
        ```

        You should see the following output

        ```{text .no-copy}
        autosplit:
                Currently enabled: no
          balancer:
                Currently enabled: no
                Currently running: no
        ```        

    2. Shut down all `mongos` nodes to stop clients from accessing the database while restore is in progress. This ensures that the final restored data doesn’t differ from the backed-up data.

    3. Shut down all `pmm-agent` and other clients that can do the write operations to the database. This is required to ensure data consistency after the restore.

    4. For PBM version 2.3.1 and earlier, manually disable point-in-time recovery if it is enabled. 

        ```{.bash data-prompt="$"}
        $ pbm config --set pitr.enabled=false
        ```

        To learn more about point-in-time recovery, see [Point-in-time recovery](../features/point-in-time-recovery.md).

=== ":material-database-refresh-outline: Physical"

    1. Shut down all `mongos` nodes as the database won't be available while the restore is in progress.
    2. Shut down all `pmm-agent` and other clients that can do the write operations to the database. This is required to ensure data consistency after the restore.
    3. Stop the arbiter nodes manually since there's no `pbm-agent` on these nodes to do that automatically.
    4. For PBM version 2.3.1 and earlier, manually disable point-in-time recovery if it is enabled. 

        ```{.bash data-prompt="$"}
        $ pbm config --set pitr.enabled=false
        ```

        To learn more about point-in-time recovery, see [Point-in-time recovery](../features/point-in-time-recovery.md).

=== ":material-select-multiple: Selective"

    You can restore a specific database or a collection either from a full or a selective backup. Read about [known limitations of selective restores](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

=== ":simple-databricks: Incremental"

    Before you start, shut down all `mongos` nodes, `pmm-agent` processes and clients that can do writes to the database as it won’t be available while the restore is in progress.

=== ":material-database-export: Snapshot-based"

    1. Shut down all `mongos` nodes. If you have set up the automatic restart of the database, disable it.
    2. Stop the arbiter nodes manually since there’s no `pbm-agent` on these nodes to do that automatically.

## Restore a database

=== ":material-data-matrix: Logical"

    1. List the backups to restore from

        ```{.bash data-prompt="$"}
        $ pbm list
        ```

    2. Restore from a desired backup. Replace the `<backup_name>` with the desired backup in the following command:

        ```{.bash data-prompt="$"}
        $ pbm restore <backup_name>
        ```

    Note that you can restore a sharded backup only into a sharded environment. It can be your existing cluster or a new one. To learn how to restore a backup into a new environment, see [Restoring a backup into a new environment](#restoring-a-backup-into-a-new-environment).

    **Post-restore steps**

    After a cluster’s restore is complete, do the following:

    1. Start the balancer and all `mongos` nodes to reload the sharding metadata.
    2. We recommend to make a fresh backup to serve as the new base for future restores.
    3. [Enable point-in-time recovery](../features/point-in-time-recovery.md#enable-point-in-time-recovery) if required.


    ### Adjust memory consumption

    Percona Backup for MongoDB config includes the restore options to adjust the memory consumption by the `pbm-agent` in environments with tight memory bounds. This allows preventing out of memory errors during the restore operation.

    ```yaml
    restore:
      batchSize: 500
      numInsertionWorkers: 10
    ```

    The default values were adjusted to fit the setups with the memory allocation of 1GB and less for the agent.

    Starting with version 2.8.0, you can override the number of insertion workers for a specific restore operation. 

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name> --numInsertionWorkers 15
    ```

    Increasing the number may increase the restore speed. However, increase the number of workers with caution, not to run into higher than expected disk and CPU usage.


    ### Restore from a logical backup made on previous major version of Percona Server for MongoDB

    In some cases you may need to restore from a backup made on previous major version of Percona Server for MongoDB. To make this happen, [Feature Compatibility Version (FCV)](https://www.mongodb.com/docs/manual/reference/command/setFeatureCompatibilityVersion/) values in both backup and the destination environment must match.

    Starting with version 2.1.0, Percona Backup for MongoDB stores the FCV value in the backup metadata. If it doesn't match the FCV value on the destination environment, you see the warning in the [`pbm status`](../reference/pbm-commands.md#pbm-status) output so that you can manually adjust it before the restore.

    ```{.bash .no-copy}
    2023-04-10T10:48:54Z 302.80KB <logical> [ERROR: backup FCV "6.0" is incompatible with the running mongo FCV "5.0"] [2023-04-10T10:49:14Z]
    2023-04-10T08:40:10Z 172.25KB <logical> [ERROR: backup mongo version "6.0.5-4" is incompatible with the running mongo version "5.0.15-13"] [2023-04-10T08:40:28Z]
    ```

    The following example illustrates the restore from a backup made on Percona Server for MongoDB 4.4 on Percona Server for MongoDB 5.0.

    1. Check the FCV value for the backup

        ```{.bash data-prompt="$"}
        $ pbm status
        ```

        Sample output:

        ```{.bash .no-copy}
        Snapshots:
        2023-04-10T10:51:28Z 530.73KB <logical> [ERROR: backup FCV "4.4" is incompatible with the running mongo FCV "5.0"] [2023-04-10T10:51:44Z]
        ```

    2. Set the Feature Compatibility Version value to 4.4

        ```{.javascript data-prompt=">"}
        > db.adminCommand( { setFeatureCompatibilityVersion: "4.4" } )
        ```

    3. Restore the database

        ```{.bash data-prompt="$"}
        $ pbm restore 2023-04-10T10:51:28Z
        ```

    4. Set the Feature Compatibility Version value to 5.0

        ```{.javascript data-prompt=">"}
        > db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )
        ```

=== ":material-database-refresh-outline: Physical"

    1. List the backups

        ```{.bash data-prompt="$"}
        $ pbm list
        ```

    2. Make a restore

        ```{.bash data-prompt="$"}
        $ pbm restore <backup_name>
        ```

    During the physical restore, `pbm-agent` processes stop the `mongod` nodes, clean up the data directory and copy the data from the storage onto every node. During this process, the database is restarted several times.

    You can [track the restore progress](restore-progress.md) using the `pbm describe-restore` command. Don't run any other commands since they may interrupt the restore flow and cause the issues with the database.

    **Post-restore steps**

    After the restore is complete, do the following:

    1. Remove the contents of the datadir on any arbiter nodes
    2. Restart all `mongod` nodes

        !!! note

            You may see the following message in the `mongod` logs after the cluster restart:

            ```{.text .no-copy}
            "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
            ```

            This is expected behavior of periodic checks upon the database start. During the restore, the `config.system.sessions` collection is dropped but Percona Server for MongoDB recreates it eventually. It is a normal procedure. No action is required from your end.

    3. Restart all `pbm-agents`

    4. Run the following command to resync the backup list with the storage:

        ```{.bash data-prompt="$"}
        $ pbm config --force-resync
        ```

    5. Start the balancer and start `mongos` nodes.

    6. We recommend to make a fresh backup to serve as the new base for future restores
    7. [Enable point-in-time recovery](../features/point-in-time-recovery.md#enable-point-in-time-recovery) if required


    ### Define a `mongod` binary location

    !!! admonition "Version added: 2.0.4"

    During physical restores, Percona Backup for MongoDB performs several restarts of the database. By default, it uses the location of the `mongod` binaries from the `$PATH` variable to access the database. If you have defined the custom path to the `mongod` binaries, make Percona Backup for MongoDB aware of it by specifying this path in the configuration file:

    ```yaml
    restore:
        mongodLocation: /path/to/mongod
    ```

    If you have different paths to `mongod` binaries on every node of your cluster / replica set, use the `mongodLocationMap` option to specify your custom paths for each node.

    ```yaml
    restore:
        mongodLocationMap:
           "node01:27017": /path/to/mongod
           "node03:27017": /another/path/to/mongod
    ```
    When running in Docker, include Percona Backup for MongoDB files together with your MongoDB binaries. See [Run Percona Backup for MongoDB in a Docker container](https://docs.percona.com/percona-backup-mongodb/install/docker.html) for more information.
    
    ### Parallel data download

    !!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

    Percona Backup for MongoDB downloads data chunks from the S3 storage concurrently during physical restore. Read more about benchmarking results in the [Speeding up MongoDB restores in PBM](https://www.percona.com/blog/speeding-up-database-restores-in-pbm) blog post by *Andrew Pogrebnoi*.

    Here's how it works:

    During the physical restore, Percona Backup for MongoDB starts the workers. The number of workers equals to the number of CPU cores by default. Each worker has a memory buffer allocated for it. The buffer is split into spans for the size of the data chunk. The worker acquires the span to download a data chunk and stores it into the buffer. When the buffer is full, the worker waits for the free span to continue the download.   

    You can fine-tune the parallel download depending on your hardware resources and database load. Edit the PBM configuration file and specify the following settings:

    ```yaml
    restore:
       numDownloadWorkers: <int>
       maxDownloadBufferMb: <int>
       downloadChunkMb: 32
    ```

    * `numDownloadWorkers` - the number of workers to download data from the storage. By default, it equals to the number of CPU cores
    * `maxDownloadBufferMb` - the maximum size of memory buffer to store the downloaded data chunks for decompression and ordering. It is calculated as `numDownloadWorkers * downloadChunkMb * 16`
    * `downloadChunkMb` is the size of the data chunk to download (by default, 32 MB)


=== ":material-select-multiple: Selective"

    1. List the backups

        ```{.bash data-prompt="$"}
        $ pbm list
        ```
    2. Run the ``pbm restore`` command in the format:

        ```{.bash data-prompt="$"}
        $ pbm restore <backup_name> --ns <database.collection>
        ```

    You can specify several namespaces as a comma-separated list for the `--ns` flag: `<db1.col1>, <db2.*>`.

    During the restore, Percona Backup for MongoDB retrieves the file for the specified database / collection and restores it.

    ### Restore with users and roles

    To restore a [custom database with users and roles](../features/selective-backup.md#restore-a-database-with-users-and-roles) from a full backup, add the `--with-users-and-roles` flag to the `pbm restore` command:

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name> --ns <database.*> --with-users-and-roles
    ```

=== ":simple-databricks: Incremental"

    Restore flow from an incremental backup is the same as the restore from a full physical backup: specify the backup name for the `pbm restore` command:

    ```{.bash data-prompt="$"}
    $ pbm restore 2022-11-25T14:13:43Z
    ```

    Percona Backup for MongoDB recognizes the backup type, finds the base incremental backup, restores the data from it and then restores the modified data from applicable incremental backups.

    After the restore is complete, do the following:

    1. Restart all `mongod` nodes and `pbm-agents`.

        !!! note

            You may see the following message in the `mongod` logs after the cluster restart:

            ```{.text .no-copy}
            "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
            ```

            This is expected behavior of periodic checks upon the database start. During the restore, the `config.system.sessions` collection is dropped but Percona Server for MongoDB recreates it eventually. It is a normal procedure. No action is required from your end.

    2. Resync the backup list from the storage.
    3. Start the balancer and the `mongos` node.
    4. As the general recommendation, make a new base backup to renew the starting point for subsequent incremental backups.

=== ":material-database-export: Snapshot-based"

     See [snapshot-based backups](../features/snapshots.md#restore-a-backup).


## Restoring a backup into a new environment

To restore a backup from one environment to another, ensure the following:

1. Percona Backup for MongoDB configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store. Once you run [`pbm list`](../reference/pbm-commands.md#pbm-list) and see the backups made from the original environment, then you can run the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command.

2. Don't run [`pbm backup`](../reference/pbm-commands.md#pbm-backup) from the new environment while Percona Backup for MongoDB configuration is pointing to the remote storage location of the original environment.

## Restoring into a cluster / replica set with a different name

Starting with version [1.8.0](../release-notes/1.8.0.md), you can restore [logical backups](../features/logical.md) into a new environment that has the same or more number of shards and these shards have different replica set names.
Starting with version [2.2.0](../release-notes/2.2.0.md), you can restore environments that have [custom shard names](https://www.mongodb.com/docs/manual/reference/command/addShard/#mongodb-dbcommand-dbcmd.addShard).

Starting with version [2.2.0](../release-notes/2.2.0.md), you can restore [physical](../features/physical.md) and [incremental physical](../features/incremental-backup.md) backups into a new environment with a different replica set names. Note that **the number of shards must be the same** as in the environment where the you made the backup.

To restore data to the environment with different replica set names, configure the name mapping between the source and target environments. You can either set the `PBM_REPLSET_REMAPPING` environment variable for `pbm` CLI or use the `--replset-remapping` flag for PBM commands. The mapping format is `<rsTarget>=<rsSource>`.

!!! important

    Configure replica set name mapping for all shards in your cluster. Otherwise, Percona Backup for MongoDB attempts to restore the unspecified shard to the target shard with the same name. If there is no shard with such name or it is already mapped to another source shard, the restore fails.

Configure the replica set name mapping:


=== ":material-application-variable-outline: Using the environment variable for `pbm` CLI in your shell"

    ```{.bash data-prompt="$"}
    $ export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"
    ```

=== ":material-console: Using the command line"

    ```{.bash data-prompt="$"}
    $ pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"
    ```

    The `--replset-remapping` flag is available for the following commands: `pbm restore`, `pbm list`, `pbm status`, `pbm oplog-replay`.

!!! note

    Follow the post-restore steps on the new environment after the restore is complete.

This ability to restore data to clusters with different replica set names and the number of shards extends the set of environments compatible for the restore.

## Next steps

[Point-in-time recovery](../usage/pitr-tutorial.md)

## Useful links

* [View restore progress](../usage/restore-progress.md)

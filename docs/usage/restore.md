# Restore a backup

!!! warning 

    Backups made with Percona Backup for MongoDB prior to v1.5.0 are incompatible for restore with Percona Backup for MongoDB v1.5.0 and later. This is because processing of system collections `Users` and `Roles` has changed: in v1.5.0, `Users` and `Roles` are copied to temporary collection during backup and must be present in the backup during restore. In earlier versions of Percona Backup for MongoDB, `Users` and `Roles` are copied to a temporary collection during restore. Therefore, restoring from these backups with Percona Backup for MongoDB v1.5.0 isn’t possible.

    The recommended approach is to make a fresh backup after upgrading Percona Backup for MongoDB to version 1.5.0.

To restore a backup, use the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command supplying the backup name from which you intend to restore. Percona Backup for MongoDB identifies the type of the backup (physical, logical or [incremental](../features/incremental-backup.md)) and restores the database up to the [restore_to_time](../reference/glossary.md#completion-time) timestamp (available in `pbm list` output starting with version 1.4.0).

## Considerations

=== "Logical"

    1. Percona Backup for MongoDB is designed to be a full-database restore tool. As of version <=1.x, it performs a full all-databases, all collections restore and does not offer an option to restore only a subset of collections in the backup, as MongoDB’s `mongodump` tool does. But to avoid surprising `mongodump` users, as of versions 1.x, Percona Backup for MongoDB replicates `mongodump’s` behavior to only drop collections in the backup. It does not drop collections that are created new after the time of the backup and before the restore. Run a `db.dropDatabase()` manually in all non-system databases (these are all databases except “local”, “config” and “admin”) before running `pbm restore` if you want to guarantee that the post-restore database only includes collections that are in the backup.

    2. While the restore is running, prevent clients from accessing the database. The data will naturally be incomplete while the restore is in progress, and writes the clients make cause the final restored data to differ from the backed-up data.

    3. If you enabled [Point-in-time recovery](../features/point-in-time-recovery.md), disable it before running `pbm restore`. This is because Point-in-Time Recovery oplog slicing and restore are incompatible operations and cannot be run together.

=== "Physical"

    1. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
    2. Physical restores are not supported for deployments with arbiter nodes.

=== "Incremental"

    1. This is a [tech preview feature](../reference/glossary.md#technical-preview-feature). We recommend using it only for testing purposes. 

    2. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
    3. Physical restores are not supported for deployments with arbiter nodes.

## Before you start

=== "Logical"

    1. Stop the balancer.

    2. Shut down all `mongos` nodes to stop clients from accessing the database while restore is in progress. This ensures that the final restored data doesn’t differ from the backed-up data.

    3. Disable point-in-time recovery if it is enabled. To learn more about point-in-time recovery, see [Point-in-time recovery](../features/point-in-time-recovery.md).

=== "Physical"

    Shut down all `mongos` nodes as the database won't be available while the restore is in progress. 

=== "Selective"
    
    You can restore a specific database or a collection either from a full or a selective backup. Read about [known limitations of selective restores](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

=== "Incremental"

    Before you start, shut down all `mongos` nodes as the database won’t be available while the restore is in progress.

## Make a restore

=== "Logical"

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

=== "Physical"

    1. List the backups 

        ```{.bash data-prompt="$"}
        $ pbm list
        ```

    2. Make a restore

        ```{.bash data-prompt="$"}
        $ pbm restore <backup_name>
        ```

    During the physical restore, `pbm-agent` processes stop the `mongod` nodes, clean up the data directory and copy the data from the storage onto every node.

    **Post-restore steps**

    After the restore is complete, do the following:

    1. Restart all `mongod` nodes

    2. Restart all `pbm-agents`

    3. Run the following command to resync the backup list with the storage:

        ```{.bash data-prompt="$"}
        $ pbm config --force-resync
        ``` 

    4. Start the balancer and start `mongos` nodes.

    5. Make a fresh backup to serve as the new base for future restores. 

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

    ### Parallel data download

    !!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

    Percona Backup for MongoDB downloads data chunks from the S3 storage concurrently during physical restore. Read more about benchmarking results in the [Speeding up MongoDB restores in PBM]() blog post by *Andrew Pogrebnoi*.

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


=== "Selective"

    1. List the backups 

        ```{.bash data-prompt="$"}
        $ pbm list
        ```
    2. Run the ``pbm restore`` command in the format:

        ```{.bash data-prompt="$"}
        $ pbm restore <backup_name> --ns <database.collection>
        ```

    During the restore, Percona Backup for MongoDB retrieves the file for the specified database / collection and restores it.

=== "Incremental"

    Restore flow from an incremental backup is the same as the restore from a full physical backup: specify the backup name for the `pbm restore` command:

    ```{.bash data-prompt="$"}
    $ pbm restore 2022-11-25T14:13:43Z
    ```

    Percona Backup for MongoDB recognizes the backup type, finds the base incremental backup, restores the data from it and then restores the modified data from applicable incremental backups.

    After the restore is complete, do the following:

    1. Restart all `mongod` nodes and `pbm-agents`. 
    2. Resync the backup list from the storage. 
    3. Start the balancer and the `mongos` node.
    4. As the general recommendation, make a new base backup to renew the starting point for subsequent incremental backups.

## Restoring a backup into a new environment

To restore a backup from one environment to another, consider the following key points about the destination environment:

* For physical restore, replica set names (both the config servers and the shards) in your new destination cluster and in the cluster that was backed up must be exactly the same.

* Percona Backup for MongoDB configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store. Once you run **pbm list** and see the backups made from the original environment, then you can run the **pbm restore** command.

Of course, make sure not to run **pbm backup** from the new environment whilst the Percona Backup for MongoDB config is pointing to the remote storage location of the original environment.

## Restoring into a cluster / replica set with a different name

Starting with version 1.8.0, you can restore **logical backups** into a new environment that has the same or more number of shards and these shards have different replica set names.

To restore data to the environment with different replica set names, configure the name mapping between the source and target environments. You can either set the `PBM_REPLSET_REMAPPING` environment variable for `pbm` CLI or use the `--replset-remapping` flag for PBM commands. The mapping format is `<rsTarget>=<rsSource>`.

!!! important

    Configure replica set name mapping for all shards in your cluster. Otherwise, Percona Backup for MongoDB attempts to restore the unspecified shard to the target shard with the same name. If there is no shard with such name or it is already mapped to another source shard, the restore fails.

Configure the replica set name mapping:


=== "Using the environment variable for `pbm` CLI in your shell"

    ```{.bash data-prompt="$"}
    $ export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"
    ``` 

=== "Using the command line"

    ```{.bash data-prompt="$"}
    $ pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"
    ```

    The `--replset-remapping` flag is available for the following commands: `pbm restore`, `pbm list`, `pbm status`, `pbm oplog-replay`.

!!! note 

    Don’t forget to make a fresh backup on the new environment after the restore is complete.

This ability to restore data to clusters with different replica set names and the number of shards extends the set of environments compatible for the restore.

## Next steps

[Point-in-time recovery](../usage/pitr-tutorial.md)

## Useful links 

* [View restore progress](../usage/restore-progress.md)




  




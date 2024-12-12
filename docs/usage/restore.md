# Restore from a logical backup

--8<-- "restore-intro.md"

## Considerations

1. While the restore is running, prevent clients from accessing the database. The data will naturally be incomplete while the restore is in progress, and writes the clients make cause the final restored data to differ from the backed-up data.

2. For versions 2.3.1 and earlier, disable [Point-in-time recovery](../features/point-in-time-recovery.md) before running `pbm restore`. This is because Point-in-Time recovery oplog slicing and restore are incompatible operations and cannot be run together.

3. Backups made with Percona Backup for MongoDB prior to v1.5.0 are incompatible for restore with Percona Backup for MongoDB v1.5.0 and later. This is because processing of system collections `Users` and `Roles` has changed: in v1.5.0, `Users` and `Roles` are copied to temporary collection during backup and must be present in the backup during restore. In earlier versions of Percona Backup for MongoDB, `Users` and `Roles` are copied to a temporary collection during restore. Therefore, restoring from these backups with Percona Backup for MongoDB v1.5.0 isn’t possible.

    The recommended approach is to make a fresh backup after upgrading Percona Backup for MongoDB to version 1.5.0.

4. For versions earlier than 1.x, Percona Backup for MongoDB performs a full all-databases, all collections restore and does not offer an option to restore only a subset of collections in the backup, as MongoDB’s `mongodump` tool does. 

5. Starting with versions 1.x, Percona Backup for MongoDB replicates `mongodump’s` behavior to only drop collections in the backup. It does not drop collections that are created new after the time of the backup and before the restore. Run a `db.dropDatabase()` manually in all non-system databases (these are all databases except “local”, “config” and “admin”) before running `pbm restore` if you want to guarantee that the post-restore database only includes collections that are in the backup.
    
## Before you start

1. Stop the balancer and disable chunks autosplit. To verify that both are disabled, run the following command:

    ```{.javascript data-prompt=">"}
    > sh.status() 
    ```

    You should see the following output:

    ```{text .no-copy}
    autosplit:
            Currently enabled: no
      balancer:
            Currently enabled: no
            Currently running: no
    ```        

2. Shut down all `mongos` nodes to stop clients from accessing the database while restore is in progress. This ensures that the final restored data doesn’t differ from the backed-up data.

3. Shut down all `pmm-agent` and other clients that can do the write operations to the database. This is required to ensure data consistency after the restore.

4. For PBM version 2.3.1 and earlier, manually disable point-in-time recovery if it is enabled. To learn more about point-in-time recovery, see [Point-in-time recovery](../features/point-in-time-recovery.md).

## Restore a database

1. List the backups to restore from

    ```{.bash data-prompt="$"}
    $ pbm list
    ```


2. Restore from a desired backup. Replace the `<backup_name>` with the desired backup in the following command:

       ```{.bash data-prompt="$"}
       $ pbm restore <backup_name>
       ```

    Note that you can restore a sharded backup only into a sharded environment. It can be your existing cluster or a new one. To learn how to restore a backup into a new environment, see [Restoring a backup into a new environment](#restoring-a-backup-into-a-new-environment).

### Post-restore steps

After a cluster’s restore is complete, do the following:

1. Start the balancer and all `mongos` nodes to reload the sharding metadata. 
2. We recommend to make a fresh backup to serve as the new base for future restores. 
3. Point-in-time recovery is re-enabled automatically upon backup completion. Otherwise, [enable point-in-time recovery](../features/point-in-time-recovery.md#enable-point-in-time-recovery) to be able to restore to a specific time.


## Adjust memory consumption

Starting with version 1.3.2, Percona Backup for MongoDB config includes the restore options to adjust the memory consumption by the `pbm-agent` in environments with tight memory bounds. This allows preventing out of memory errors during the restore operation.

```yaml
restore:
  batchSize: 500
  numInsertionWorkers: 10
```

The default values were adjusted to fit the setups with the memory allocation of 1GB and less for the agent.

!!! note 

    Starting with version 2.8.0, you can override the number of insertion workers for a specific restore operation. 

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name> --numInsertionWorkers 15
    ```

    Increasing the number may increase the restore speed. However, increase the number of workers with caution, not to run into higher than expected disk and CPU usage.

## Restore from a logical backup made on previous major version of Percona Server for MongoDB

In some cases you may need to restore from a backup made on previous major version of Percona Server for MongoDB. To make this happen, [Feature Compatibility Version (FCV) :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/command/setFeatureCompatibilityVersion/) values in both backup and the destination environment must match. 

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

<<<<<<< HEAD
4. Set the Feature Compatibility Version value to 5.0
=======
    ### Restore a collection under a different name

    You can restore a specific collection under a different name alongside the current collection. This is useful when you troubleshoot database issues and need to compare the data in both collections to identify the root of the issue.

    Note that in version 2.8.0 you can restore a single collection and this collection must be unsharded. 

    To restore a collection, pass the collection name from the backup for the `--ns-from` flag and the new name for the `--ns-to` flag:

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name> --ns-from <database.collection> --ns-to <database.collection_new>
    ```

    The new collection has the same data and indexes as the source collection. You must provide a unique name for the collection you restore, otherwise the restore fails.

    You can restore a collection under a new name up to the specified time. Instead of the backup name, specify the timestamp, the source collection name and the new name as follows:

    ```{.bash data-prompt="$"}
    $ pbm restore --time=<timestamp> --ns-from <database.collection> --ns-to <database.collection_new>
    ```


=== ":simple-databricks: Incremental"

    Restore flow from an incremental backup is the same as the restore from a full physical backup: specify the backup name for the `pbm restore` command:
>>>>>>> 3d406df... PBM-1427 Documented how to clone collections (#225)

    ```{.javascript data-prompt=">"}
    > db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )
    ```

## Next steps

[Point-in-time recovery](../usage/pitr-tutorial.md)

## Useful links 

* [View restore progress](../usage/restore-progress.md)




  




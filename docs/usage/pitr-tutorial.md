# Make a point-in-time restore

## Preconditions

Run [`pbm status`](../reference/pbm-commands.md#pbm-status) or [`pbm list`](../reference/pbm-commands.md#pbm-list) commands to check that the full backup snapshot exists and there are oplog slices.

## Before you start

1. Disable point-in-time recovery. A restore and point-in-time recovery oplog slicing are incompatible operations and cannot be run simultaneously. 

    ```{.bash data-prompt="$"}
    $ pbm config --set pitr.enabled=false
    ```

2. Stop the balancer and `mongos` nodes.
3. Make sure no writes are made to the database during restore. 

## Procedure

=== "From logical backups" 

    Run [`pbm restore`](../reference/pbm-commands.md#pbm-restore) and specify the timestamp from the valid range:    

    ```{.bash data-prompt="$"}
    $ pbm restore --time="2022-12-14T14:27:04"
    ```    

    The timestamp you specify for the restore must be within the time ranges in the PITR section of `pbm list` output. Percona Backup for MongoDB automatically selects the most recent backup among logical, physical and incremental in relation to the specified timestamp and uses that as the base for the restore.    

    To illustrate this behavior, let’s use the following `pbm list` output as the example.     

    ```{.bash .no-copy}
    $ pbm list    

      2021-08-04T13:00:58Z [restore_to_time: 2021-08-04T13:01:23Z]
      2021-08-05T13:00:47Z [restore_to_time: 2021-08-05T13:01:11Z]
      2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09Z]
      2021-08-06T08:03:43Z [restore_to_time: 2021-08-06T08:04:08Z]
      2021-08-06T08:18:17Z [restore_to_time: 2021-08-06T08:18:41Z]    

    PITR <off>:
      2021-08-04T13:01:24 - 2021-08-05T13:00:11
      2021-08-06T08:03:10 - 2021-08-06T08:18:29
      2021-08-06T08:18:42 - 2021-08-06T08:33:09
    ```    

    For timestamp `2021-08-06T08:10:10`, the backup snapshot `2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09]` is used as the base for the restore as it is the most recent one.    

    If you [select a backup snapshot for the restore with the `–-base-snapshot` option](../features/point-in-time-recovery.md#select-a-backup-snapshot-for-the-restore), the timestamp for the restore must also be later than the selected backup.    

    !!! admonition "See also"    

        [Restore a backup](restore.md)    

    ### Post-restore steps    

    A restore operation changes the time line of oplog events. Therefore, all oplog slices made after the restore time stamp and before the last backup become invalid. After the restore is complete, do the following:    

    1. Make a new backup to serve as the starting point for oplog updates:    

        ```{.bash data-prompt="$"}
        $ pbm backup
        ```    

    2. Re-enable point-in-time recovery to resume saving oplog slices:    

        ```{.bash data-prompt="$"}
        $ pbm config --set pitr.enabled=true
        ```

=== "From physical backups"

    Starting with version [2.2.0](../release-notes/2.2.0.md), you can recover your database from a full or an incremental physical backup in the same automated fashion as from a logical one. Percona Backup for MongoDB restores the backup snapshot and automatically replays the oplog events on top of it up to the specified time, guaranteeing data consistency. This helps you prevent data loss during a disaster and gives you the same user experience when managing backups and restores.    

    To restore a database from a physical backup, specify the time for the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command:    

    ```{.bash data-prompt="$"}
    $ pbm restore --time <timestamp> -w
    ```    

    Percona Backup for MongoDB recognizes if it is a full or an incremental backup and restores the database from it up to the specified time.     

    !!! note    

        For PBM versions earlier then 2.3.0, the command for the point-in-time recovery is the following:
        
        ```{.bash data-prompt="$"}
        $ pbm restore --base-snapshot=<backup_name> --time <timestamp> -w 
        ```

        The `--base-snapshot` flag is required. Otherwise, PBM will look for a logical backup even if there is none or there is a more recent physical backup.    

    After the point-in-time recovery is complete, perform these post-restore steps:    

    1. Restart all `mongod` nodes.    

    2. Restart all `pbm-agents`.    

    3. Resync the backup list with the storage:    

        ```{.bash data-prompt="$"}
        $ pbm config --force-resync
        ```    

    4. Start the balancer and start `mongos` nodes.    

    5. Make a fresh backup to serve as the new base for future restores.    

    6. [Enable point-in-time routine](../features/point-in-time-recovery.md#enable-point-in-time-recovery) to resume saving oplog slices.    

    For Percona Backup for MongoDB version 2.1.0 and earlier, point-in-time recovery consists of the following steps:    

    * Restore from the physical backup snapshot.
    * Manual replay of oplog events on top of this snapshot up to a specific timestamp.    

    For how to replay oplog events on top of a backup, see [Oplog replay for physical backups](oplog-replay.md#oplog-replay-for-physical-backups).

    ### Implementation specifics

    1. Due to the physical restore logic and flow, PBM replays oplog events on the primary node of every shard when Percona Server for MongoDB is shut down. After the database start, the remaining nodes receive the data during the initial sync.
    2. When doing point-in-time recovery for deployments with sharded collections, PBM only writes data to existing ones and doesn’t support creating new collections. Therefore, whenever you create a new sharded collection, make a new backup for it to be included there.

## Select a backup snapshot for the restore

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

You can recover your database to the specific point in time using any backup snapshot, and not only the most recent one. Run the `pbm restore` command with the `--base-snapshot=<backup_name>` flag where you specify the desired backup snapshot.

To restore from any backup snapshot, Percona Backup for MongoDB requires continuous oplog. After the backup snapshot is made and point-in-time recovery is re-enabled, it copies the oplog saved with the backup snapshot and creates oplog slices from the end time of the latest slice to the new starting point thus making the oplog continuous.


## Restore selected databases and collections

!!! important

    Supported only for replica sets.
    Available for logical backups.

1. Before you start:

    1. Read [known limitations for selective backups and restores](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).
    2. Check that you [have made a full backup](start-backup.md#make-a-backup) because it serves as the base for point-in-time recovery. Any selective backup is ignored.

2. To restore the desired database or a collection to a point in time, run the ``pbm restore`` command as follows:

    ```{.bash data-prompt="$"}
    $ pbm restore --base-snapshot <backup_name> --time <timestamp> \
    --ns <db.collection>
    ```

    You can specify the selective backup as the base snapshot for the Point-in-time restore. In this case, Percona Backup for MongoDB restores only the namespace(s) included in this backup to the specified time.    

    Alternatively, you can use a full backup snapshot and restore the desired namespaces (databases or collections) up to the specific time from it. Specify them as the comma-separated list for the `pbm restore` command.    

    When point-in-time recovery is started, Percona Backup for MongoDB uses the provided base snapshot, restores the specified namespace(s) and replays oplog on top of it up to the specified time. If no base snapshot is provided, Percona Backup for MongoDB uses the most recent full backup snapshot.

## Useful links

* [Restore a backup](restore.md)
* [Replay oplog from arbitrary start time](oplog-replay.md)



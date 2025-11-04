# Point-in-time restore from physical backups

--8<-- "pitr-preparation.md"

## Procedure

!!! admonition "Version added: [2.2.0](../release-notes/2.2.0.md)"

You can recover your database from a full or an incremental physical backup in the same automated fashion as from a logical one. Percona Backup for MongoDB restores the backup snapshot and automatically replays the oplog events on top of it up to the specified time, guaranteeing data consistency. This helps you prevent data loss during a disaster and gives you the same user experience when managing backups and restores.    

To restore a database from a physical backup, specify the time for the [`pbm restore`](../reference/pbm-commands.md#pbm-restore) command:    

```bash
pbm restore --time <timestamp> 
```    

Percona Backup for MongoDB recognizes if it is a full or an incremental backup and restores the database from it up to the specified time.     

You can [track the restore progress](restore-progress.md) using the `pbm describe-restore` command. Don't run any other commands since they may interrupt the restore flow and cause the issues with the database.


!!! note    

    For PBM versions earlier than 2.3.0, the command for the point-in-time recovery is the following:
        
    ```bash
    pbm restore --base-snapshot=<backup_name> --time <timestamp>
    ```

    The `--base-snapshot` flag is required. Otherwise, PBM will look for a logical backup even if there is none or there is a more recent physical backup.    

### Post-restore steps

After the point-in-time recovery is complete, perform these post-restore steps:   

1. Restart all `mongod` nodes.    

2. Restart all `pbm-agents`.    

3. Resync the backup list with the storage:    

    ```bash
    pbm config --force-resync
    ```    

4. Start the balancer and start `mongos` nodes.    
5. Make a fresh backup to serve as the new base for future restores.    

6. [Enable point-in-time routine](../features/point-in-time-recovery.md#enable-point-in-time-recovery) to resume saving oplog slices.    

??? admonition "Percona Backup for MongoDB version 2.1.0 and earlier"

    For Percona Backup for MongoDB version 2.1.0 and earlier, point-in-time recovery consists of the following steps:    

    * Restore from the physical backup snapshot.
    * Manual replay of oplog events on top of this snapshot up to a specific timestamp.    

    For how to replay oplog events on top of a backup, see [Oplog replay for physical backups](oplog-replay.md#oplog-replay-for-physical-backups).

### Implementation specifics

1. Due to the physical restore logic and flow, PBM replays oplog events on the primary node of every shard when Percona Server for MongoDB is shut down. After the database start, the remaining nodes receive the data during the initial sync.
2. When doing point-in-time recovery for deployments with sharded collections, PBM only writes data to existing ones and doesn’t support creating new collections. Therefore, whenever you create a new sharded collection, make a new backup for it to be included there.


## Useful links

* [Restore a backup](restore.md)
* [Replay oplog from arbitrary start time](oplog-replay.md)



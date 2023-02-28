# Logical backups and restores

*Logical* backup is the copying of the actual database data. A `pbm-agent` connects to the database, retrieves the data and writes it to the remote backup storage. 

Logical restore is the reverse process: the ``pbm-agent`` retrieves the backup data from the storage and inserts it on every primary node in the cluster. The remaining nodes receive the data during the replication process.

Logical backups allow for point in time recovery. 

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
| - Easy to operate with, using a single command <br> - Support for point-in-time recovery <br> - The backup size is smaller as it includes only the data | - Much slower than physical backup / restore <br> - Adds database overhead on reading and inserting the data| Sharded clusters and non-sharded replica sets | 

## Preparation

[Install](installation.md) and [set up PBM](initial-setup.md).

## Make a backup

To make a logical backup, run the `pbm backup` command:

```
pbm backup
```

To learn more about making backups, see [Start a backup](usage/start-backup.md)

## Make a restore

To restore a database in full from a backup, do the following:

1. Complete all [preconditions for the restore](usage/restore.md#preconditions-for-the-restore-in-sharded-clusters)
3. View completed backups:

    ```
    pbm list
    ```

3. Restore from the desired backup. Replace the `<backup_name>` with the desired backup in the following command:

	```
	pbm restore <backup_name>
	```

4. After a cluster’s restore is complete, restart all `mongos` nodes to reload the sharding metadata.

For more information about restores, read [Restore a backup](usage/restore.md). 

To restore a subset of data, refer to [Selective restore](usage/selective-backup.md#selective-restore)

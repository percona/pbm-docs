# Make a logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! warning

    Sharded time series collections are not supported. If you use them in your deployment, you won't be able to make a backup. 
 

To make a backup, run the following command:

```{.bash data-prompt="$"}
$ pbm backup --type=logical
```
     
Logical backup is the default one so you can bypass the `--type` flag. 

During *logical* backups, Percona Backup for MongoDB copies the actual data to the backup storage.

Starting with version 2.0.0, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is now the default data format since it allows [selective restore](restore.md). Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB.




=== ":material-database-refresh-outline: Physical"
     
    !!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

     ```{.bash data-prompt="$"}
     $ pbm backup --type=physical
     ```

     During a *physical* backup, Percona Backup for MongoDB  copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage. 
     
    !!! warning 

        During the period the backup cursor is open, database checkpoints can be created, but no checkpoints can be deleted. This may result in significant file growth.
    
     Starting with [2.4.0](../release-notes/2.4.0.md), PBM doesn't stop [point-in-time recovery oplog slicing](../features/point-in-time-recovery.md#oplog-slicing), if it's enabled, but runs it in parallel. This ensures [point-in-time recovery](pitr-tutorial.md) to any timestamp if it takes too long (e.g. hours) to make a backup snapshot.

=== ":material-select-multiple: Selective"

    !!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

    Before you start, read about [selective backups known limitations](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

    To make a selective backup,  run the `pbm backup` command and provide the value for the `--ns` flag in the format `<database.collection>`. The `--ns` flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

     ```{.bash data-prompt="$"}
     $ pbm backup --ns=staff.Payments
     ```

     To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

     ```{.bash data-prompt="$"}
     $ pbm backup --ns=Invoices.*
     ```

     During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

     Multi-format is now the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 

=== ":simple-databricks: Incremental"
    
    !!! admonition "Version added: [2.0.3](../release-notes/2.0.3.md)"

    Before you start, read more about [incremental backup](../features/incremental-backup.md#considerations).

    To start incremental backups, first make a full incremental backup. It will serve as the base for subsequent incremental backups:

    ```{.bash data-prompt="$"} 
    $ pbm backup --type incremental --base
    ```

    The `pbm-agent` starts tracking the incremental backup history to be able to calculate and save the difference in data blocks. After that you can run regular incremental backups:

    ```{.bash data-prompt="$"}
    $ pbm backup --type incremental
    ```

    The incremental backup history looks like this:

    ??? example "Sample output"

        ```{.bash .no-copy} 
        Snapshots:
            2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
            2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
            2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
            2022-11-24T14:45:53Z 220.13GB <incremental, base> [restore_to_time: 2022-11-24T14:45:55Z]
        ```

=== ":material-database-export: Snapshot-based"

    See [snapshot-based backups](../features/snapshots.md#make-a-backup).


## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


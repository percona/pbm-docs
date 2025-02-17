# Restore from a logical backup

--8<-- "restore-intro.md"

## Before you start

You can restore a specific database or a collection either from a full or a selective backup. Read about [known limitations of selective restores](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

## Restore a database

1. List the backups 

    ```{.bash data-prompt="$"}
    $ pbm list
    ```
2. Run the ``pbm restore`` command in the format:

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name> --ns <database.collection>
    ```
    
 You can specify several namespaces as a comma-separated list for the `--ns` flag: `<db1.col1>,<db2.*>`. For example, `--ns=customers.payments,invoices.*`.

During the restore, Percona Backup for MongoDB retrieves the file for the specified database / collection and restores it.

### Restore with users and roles

To restore a [custom database with users and roles](../features/selective-backup.md#restore-a-database-with-users-and-roles) from a full backup, add the `--with-users-and-roles` flag to the `pbm restore` command:

```{.bash data-prompt="$"}
$ pbm restore <backup_name> --ns <database.*> --with-users-and-roles
```

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

## Post-restore steps

After the restore is complete, do the following:

1. Start the balancer and all mongos nodes to reload the sharding metadata.
2. We recommend to make a fresh backup to serve as the new base for future restores.

## Next steps

[Point-in-time recovery](../usage/pitr-selective.md)

## Useful links 

* [View restore progress](../usage/restore-progress.md)




  




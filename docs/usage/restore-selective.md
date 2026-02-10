# Make a selective restore from a logical backup

--8<-- "restore-intro.md"

## Before you start

You can restore a specific database or a collection either from a full or a selective backup. Read about [known limitations of selective restores](../features/known-limitations.md#selective-backups-and-restores).

## Restore a database

1. List the backups

    ```bash
    pbm list
    ```
    
2. Run the ``pbm restore`` command in the format:

    ```bash
    pbm restore <backup_name> --ns <database.collection>
    ```

 You can specify several namespaces as a comma-separated list for the `--ns` flag: `<db1.col1>,<db2.*>`. For example, `--ns=customers.payments,invoices.*`.

During the restore, Percona Backup for MongoDB retrieves the file for the specified database / collection and restores it.

### Restore with users and roles

To restore a [custom database with users and roles](../features/selective-backup.md#restore-a-database-with-users-and-roles) from a full backup, add the `--with-users-and-roles` flag to the `pbm restore` command:

```bash
pbm restore <backup_name> --ns <database.*> --with-users-and-roles
```

### Selective restore with users and roles

#### Overview

!!! warning
    - The `--with-users-and-roles` flag applies only to users and roles defined within the database being backed up or restored. Global users and roles defined at the cluster level **are not included**.
    - If applications rely on roles or privileges that span multiple databases, a selective restore of a single database may not fully reestablish access-control. Always verify dependencies before restore.
    - Restoring users and roles will overwrite existing definitions in the target database. Review current role configurations before restore to avoid accidental loss of custom changes.


Percona Backup for MongoDB allows you to perform selective restore of databases and collections. Additionally, you can choose to include **users and roles defined** in the database in your selective backup, ensuring that access control is restored along with the data.

To restore a specific namespace and include users and roles, run the following command:

```sh
pbm restore --ns="mydb.*" --with-users-and-roles <selective-backup-id>
```

where:

`--ns="mydb.*"` → Restores only the collections belonging to mydb.

`--with-users-and-roles` → Restores the database-defined users and roles alongside the data.

`<selective-backup-id>` → The identifier of the backup to restore from (as shown in Percona Backup for MongoDB backup listings and logs).

??? info "What happens under the hood?"
    - Percona Backup for MongoDB restores the selected collections within `mydb` from the specified backup.
    - Percona Backup for MongoDB restores roles where the db field matches `mydb`.
    - Percona Backup for MongoDB restores users where the db field matches `mydb`, including their role assignments within `mydb`.

**Example**

```sh
pbm restore --ns="invoices.*" --with-users-and-roles 2025-03-10T10:44:52Z
```

#### Use cases

=== "Partial restore after data loss"
    A service using `mydb` experienced accidental deletes or corruption, while other databases in the cluster remain unaffected. Selective restore limits recovery to only the required database.

=== "Roll back access control changes"
    A recent modification to custom roles in `mydb` introduced permission failures. Applications that rely on those roles can no longer perform required operations. 
    
    To ensure full recovery, you need to restore not just the data but also the users and roles tied to the database’s access-control.

### Restore a collection under a different name

You can restore a specific collection under a different name alongside the current collection. This is useful when you troubleshoot database issues and need to compare the data in both collections to identify the root of the issue.

Note that in version 2.8.0 you can restore a single collection and this collection must be unsharded.

To restore a collection, pass the collection name from the backup for the `--ns-from` flag and the new name for the `--ns-to` flag:

```bash
pbm restore <backup_name> --ns-from <database.collection> --ns-to <database.collection_new>
```

The new collection has the same data and indexes as the source collection. You must provide a unique name for the collection you restore, otherwise the restore fails.

You can restore a collection under a new name up to the specified time. Instead of the backup name, specify the timestamp, the source collection name and the new name as follows:

```bash
pbm restore --time=<timestamp> --ns-from <database.collection> --ns-to <database.collection_new>
```

## Post-restore steps

After the restore is complete, do the following:

1. Start the balancer and all mongos nodes to reload the sharding metadata.
2. We recommend to make a fresh backup to serve as the new base for future restores.

## Next steps

[Point-in-time recovery](../usage/pitr-selective.md){.md-button}

## Useful links

* [View restore progress](../usage/restore-progress.md)

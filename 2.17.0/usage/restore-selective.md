> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make a selective restore from a logical backup

To restore a backup, use the [`pbm restore`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-restore) command supplying the backup name from which you intend to restore. Percona Backup for MongoDB identifies the type of the backup (physical, logical or [incremental](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/incremental-backup.html)) and restores the database up to the [restore_to_time](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#completion-time) timestamp (available in `pbm list` output starting with version 1.4.0).

## Before you start

You can restore a specific database or a collection either from a full or a selective backup. Read about [known limitations of selective restores](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/known-limitations.html#selective-backups-and-restores).

## Restore a database with users and roles

Warning

- The `--with-users-and-roles` flag applies only to users and roles defined within the database being backed up or restored. Global users and roles defined at the cluster level **are not included**.
- If applications rely on roles or privileges that span multiple databases, a selective restore of a single database may not fully reestablish access control. Always verify dependencies before restore.
- Restoring users and roles will overwrite existing definitions in the target database. Review current role configurations before restore to avoid accidental loss of custom changes.

Percona Backup for MongoDB allows you to perform selective restore of databases and collections. Additionally, you can choose to include **users and roles defined** in the database in your selective backup, ensuring that access control is restored along with the data.

1. List the backups

   ```
   pbm list
   ```

1. To restore a specific database and include users and roles, run the following command.

   ```
   pbm restore --ns="mydb.*" --with-users-and-roles <backup-name>
   ```

   Note:

   - Starting with version [2.14.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.14.0.html), before a restore operation is executed you have to confirm the action (to bypass it, add the `-y` or `--yes` flag).
   - To restore without the users and roles, skip the `--with-users-and-roles` flag.
   - You can specify several namespaces as a comma-separated list for the `--ns` flag: `<db1.*>,<db2.*>`. For example, `--ns=customers.*,invoices.*`.
   - During the restore, Percona Backup for MongoDB retrieves the file for the specified database/collection and restores it.

   **Where:**

   - `--ns="mydb.*"` **→** Restores only the collections belonging to mydb.
   - `--with-users-and-roles` **→** Restores the database-defined users and roles alongside the data.
   - `<backup-name>` **→** The identifier of the backup to restore from (as shown in Percona Backup for MongoDB backup listings and logs). What happens under the hood?
   - Percona Backup for MongoDB restores the selected collections within `mydb` from the specified backup.
   - Percona Backup for MongoDB restores roles where the db field matches `mydb`.
   - Percona Backup for MongoDB restores users where the db field matches `mydb`, including their role assignments within `mydb`.

   Example

   ```
   pbm restore --ns="invoices.*" --with-users-and-roles 2025-03-10T10:44:52Z
   ```

   This command restores all collections in the `invoices` database, along with the users and roles defined in `invoices`, from the backup `2025-03-10T10:44:52Z`.

### Namespace requirements

The following are the namespace requirements:

1. The `--with-users-and-roles` flag requires a collection wildcard in the namespace.

   For example, `--ns="test.*"` is valid, but `--ns="test.col"` is not valid.

1. Use `--with-users-and-roles` only with selective restore (i.e., when you specify `--ns`). If you are not using `--ns`, you are not performing a selective restore, and this option is not required.

## Restore a collection under a different name

You can restore a specific collection under a different name alongside the current collection. This is useful when you troubleshoot database issues and need to compare the data in both collections to identify the root of the issue.

Note that in version 2.8.0 you can restore a single collection and this collection must be unsharded.

To restore a collection, pass the collection name from the backup for the `--ns-from` flag and the new name for the `--ns-to` flag:

```
pbm restore <backup_name> --ns-from <database.collection> --ns-to <database.collection_new>
```

The new collection has the same data and indexes as the source collection. You must provide a unique name for the collection you restore, otherwise the restore fails.

You can restore a collection under a new name up to the specified time. Instead of the backup name, specify the timestamp, the source collection name and the new name as follows:

```
pbm restore --time=<timestamp> --ns-from <database.collection> --ns-to <database.collection_new>
```

Version added: [2.14.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.14.0.html)

Before a restore operation is executed you have to confirm the action (to bypass it, add the `-y` or `--yes` flag).

## Post-restore steps

After the restore is complete, do the following:

1. Start the balancer and all mongos nodes to reload the sharding metadata.
1. We recommend to make a fresh backup to serve as the new base for future restores.

## Next steps

[Point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/pitr-selective.html)

## Useful links

- [View restore progress](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-progress.html)

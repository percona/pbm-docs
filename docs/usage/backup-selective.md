# Make a selective logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

Before you start, read about [selective backups known limitations](../features/known-limitations.md#selective-backups-and-restores).

To make a selective backup, run the `pbm backup` command and provide the value for the `--ns` flag in the format `<database.collection>`. The `--ns` flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

```bash
pbm backup --ns=customers.payments
```

To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

```bash
pbm backup --ns=invoices.*
```

To back up multiple namespaces, specify them as a comma-separated list for the `--ns` flag: `<db1.col1>`,`<db2.*>`,`<db3.collX>`. The number of namespaces to specify is unlimited.

During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 

## Selective backup with users and roles

Percona Backup for MongoDB allows you to perform selective backups and restores of databases and collections. Additionally, you can choose to include users and roles defined in the database in your selective backup, ensuring that access control is restored along with the data.


This feature is useful in the following cases:

- Migrating a database to a new environment while keeping its access control intact.

- Restoring a subset of collections along with the users and roles that manage them.

- Creating test environments with the same security model as production.

## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-selective.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


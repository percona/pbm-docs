# Make a logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

Before you start, read about [selective backups known limitations](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

To make a selective backup, run the `pbm backup` command and provide the value for the `--ns` flag in the format `<database.collection>`. The `--ns` flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

```{.bash data-prompt="$"}
$ pbm backup --ns=customers.payments
```

To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

```{.bash data-prompt="$"}
$ pbm backup --ns=invoices.*
```

To back up multiple namespaces, specify them as a comma-separated list for the `--ns` flag: `<db1.col1>`,`<db2.*>`,`<db3.collX>`. The number of namespaces to specify is unlimited.

During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 


## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


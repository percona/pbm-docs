# Make a selective logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

Before you start, read about [selective backups known limitations](../features/known-limitations.md#selective-backups-and-restores).

To make a selective backup, run the pbm backup command and provide the value for the --ns flag in the format <database.collection>. The --ns flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

```bash
pbm backup --ns=customers.payments
```
To back up the **Invoices** database and all collections that it includes, run the pbm backup command as follows:

```bash
pbm backup --ns=invoices.*
```
To back up multiple namespaces, specify them as a comma-separated list for the --ns flag: `<db1.col1>,<db2.*>,<db3.collX>`. The number of namespaces to specify is unlimited.

## Selective backup with users and roles

### Overview

Percona Backup for MongoDB enables you to perform selective backups of databases and collections. Also, you can choose to include **users and roles defined** in the database in your selective backup, ensuring that access control is restored along with the data.
 
To back up a specific namespace and include users and roles:

!!! warning
     Including users and roles (`--with-users-and-roles`) is not supported when backing up a specific collection (for example, `--ns=db.collection`). To include users and roles, you must back up the entire database by using `--ns='db.*'`.

```sh
pbm backup --ns="mydb.*" --with-users-and-roles
```

**Where:**

- `--ns="mydb.*"` **→** specifies the namespace (all collections in `mydb`).

- `--with-users-and-roles` **→** includes all users and custom roles defined in `mydb` in the backup.

??? info "What happens under the hood?"
    - Percona Backup for MongoDB captures all collections within `mydb`.
    - Percona Backup for MongoDB filters the users and roles for entities where the `db` field matches `mydb`.
    - Global administrative roles or users defined in other databases are excluded.


## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-selective.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


# Make a selective logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

Before you start, read about [selective backups known limitations](../features/known-limitations.md#selective-backups-and-restores).

## Selective backup with users and roles

### Overview

Percona Backup for MongoDB enables you to perform selective backups of databases and collections. Also, you can choose to include **users and roles defined** in the database in your selective backup, ensuring that access control is restored along with the data.

During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 

To back up a specific namespace and include users and roles, run the following command:


```sh
pbm backup --ns="mydb.*" --with-users-and-roles
```
To back up multiple namespaces, specify them as a comma-separated list for the `--ns` flag: `<db1.col1>`,`<db2.*>`,`<db3.collX>`. The number of namespaces to specify is unlimited.

where:

`--ns="mydb.*"` **→** specifies the namespace (all collections in `mydb`).

`--with-users-and-roles` **→** includes all users and custom roles defined in `mydb` in the backup.

??? info "What happens under the hood?"
    - Percona Backup for MongoDB captures all collections within `mydb`.
    - Percona Backup for MongoDB filters the users and roles for entities where the `db` field matches `mydb`.
    - Global administrative roles or users defined in other databases are excluded.


**Example**

```sh
pbm backup --ns="invoices.*" --with-users-and-roles
```

This command backs up all collections in the **invoices** database, along with its users and roles.

!!! warning
     Including users and roles (`--with-users-and-roles`) is not supported when backing up a specific collection (for example, `--ns=db.collection`). To include users and roles, you must back up the entire database by using `--ns='db.*'`.

### Use cases

=== "Partial migration of a database"
    As applications scale, migrating a specific database from a shared cluster to a dedicated cluster becomes necessary. Using the `--with-users-and-roles` flag ensures that the destination cluster inherits the application-specific users and custom roles immediately, thereby preventing errors post-migration.


=== "Staging environment"
    To reproduce production issues or validate security patches, you need a staging environment that mirrors production exactly.
    
    By backing up `mydb` together with its users and roles, the copy reflects not only the data but also the access-control model. This enables accurate reproduction of permission-related behavior such as read/write restrictions, role grants, and user privileges.


## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-selective.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


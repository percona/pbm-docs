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

### Overview

Percona Backup for MongoDB allows you to perform selective backups and restores of databases and collections. Additionally, you can choose to include **users and roles defined** in the database in your selective backup, ensuring that access control is restored along with the data.

This feature is useful in the following cases:

- Migrating a database to a new environment while keeping its access control intact.

- Restoring a subset of collections along with the users and roles that manage them.

To back up a specific namespace and include users and roles, run the following command:

```sh
pbm backup --ns="mydb.*" --with-users-and-roles
```

where:

`--ns="mydb.*"` specifies the namespace (all collections in mydb).

`--with-users-and-roles` ensures that users and roles defined in `mydb` are included in the backup.

The `--with-users-and-roles` flag ensures that any custom users and roles defined within the target database are included, maintaining the integrity of your access control list (ACL) without needing a full cluster restore.


??? info "What happens under the hood?"
    - Percona Backup for MongoDB captures all collections within `mydb`.
    - Percona Backup for MongoDB filters the users and roles for entities where the `db` field matches `mydb`.
    - Global roles or users defined in the admin database for other namespaces are excluded.

**Example**

```sh
pbm backup --ns="invoices.*" --with-users-and-roles
```

This command backs up all collections in the **invoices** database along with its users and roles.

### Use cases

=== "Partial Migration for a database"
    Organizations often start with a shared cluster hosting multiple databases for different services. Over time, scaling, performance, or compliance needs may require isolating one database into its own dedicated cluster.

=== "Roll back access control changes"
    A recent modification to custom roles in `mydb` introduced permission failures. Applications that rely on those roles can no longer perform required operations. To ensure full recovery, you need to restore not just the data but also the users and roles tied to the database’s access-control.

=== "Staging environment"
    You need a production-like copy of `mydb` in a separate environment to reproduce issues, validate fixes, or conduct testing.
    
    By backing up mydb together with its users and roles, the copy reflects not only the data but also the access-control model. This enables accurate reproduction of permission-related behavior such as read/write restrictions, role grants, and user privileges.





## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-selective.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


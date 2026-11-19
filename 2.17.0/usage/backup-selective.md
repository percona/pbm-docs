> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Make a selective logical backup

## Before you start

Before taking a backup, verify PBM is installed and running.

1. [Install](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [set up Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

1. Check that `pbm agent` is running with the [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) command

1. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them.

   To check the version, run the following commands:

   - [`pbm status`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of pbm-agents
   - [`pbm version`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-status) to check the version of PBM CLI.

## Procedure

Version added: [2.0.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.0.0.html)

Before you start, read about [selective backups known limitations](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/known-limitations.html#selective-backups-and-restores).

To make a selective backup, run the pbm backup command and provide the value for the –ns flag in the format . The –ns flag value is case sensitive. For example, to back up the “Payments” collection, run the following command:

```
pbm backup --ns=customers.payments
```

To back up the **Invoices** database and all collections that it includes, run the `pbm backup` command as follows:

```
pbm backup --ns=invoices.*
```

To back up multiple namespaces, specify them as a comma-separated list for the –ns flag: `<db1.col1>,<db2.*>,<db3.*>`. The number of namespaces to specify is unlimited.

## Selective backup with users and roles

Version added: [2.13.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.13.0.html)

### Overview

Percona Backup for MongoDB allows you to create selective backups of databases and collections, **including the users and roles** defined within the database. This ensures that access control is restored along with the data.

To back up a specific namespace and include users and roles:

Warning

Including users and roles (`--with-users-and-roles`) isn’t supported for collection-level backups (for example, `--ns="db.collection"`). To include users and roles, you must back up entire databases only, using `--ns="db.*"`. As a result, mixed patterns like `--ns="db1.*,db2.*,db3.col"` will fail because `db3.col` targets a single collection.

```
pbm backup --ns="mydb.*" --with-users-and-roles
```

**Where:**

- `--ns="mydb.*"` **→** specifies the namespace (all collections in `mydb`).
- `--with-users-and-roles` **→** includes all users and custom roles defined in `mydb` in the backup.

What happens under the hood?

- Percona Backup for MongoDB captures all collections within `mydb`.
- Percona Backup for MongoDB filters the users and roles for entities where the `db` field matches `mydb`.
- Global administrative roles or users defined in other databases are excluded.

## Next steps

[List backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/list-backup.html) [Make a restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-selective.html)

## Useful links

- [Backup and restore types](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)
- [Schedule backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/schedule-backup.html)

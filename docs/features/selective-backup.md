# Selective backup and restore

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

??? admonition "Implementation history"

    The following table lists the changes in the implementation of selective backups and the versions that introduced those changes:

    | Version            | Description        |
    | ------------------ | ------------------ |
    | [2.0.3](../release-notes/2.0.3.md) | Support for non-sharded collections in sharded clusters|
    | [2.1.0](../release-notes/2.1.0.md) | Added support for sharded collections |
    | [2.5.0](../release-notes/2.5.0.md) | Ability to restore databases with users and roles |
    | [2.8.0](../release-notes/2.8.0.md) | Ability to define multiple namespaces for backup |
    | [2.8.0](../release-notes/2.8.0.md) | Ability to restore a single non-sharded collection under a different name |

You can back up and restore certain namespaces - databases or collections. For example, if your "Payments" collection in the "Customers" database was corrupted, you can restore only this collection from your full backup. Or, if your "Invoices" database contains sensitive data and must be backed up frequently, you can configure the backup of only this database. 

Starting in version 2.8.0, you can define several databases or collections for a backup. This simplifies the backup management since instead of having backups for every namespace, you accumulate the required data within a single backup. 

Using selective backups and restores, you work only with the desired subset of data without disrupting the operations of your whole cluster. 

You also drastically reduce time on backup / restore operations of the whole data set and save on storage consumption. 

With the selective backup and restore functionality, you have the following options:
{.power-number}

1.	Back up a single database or a specific collection and restore all data from it. 
2.  Back up certain databases and/or collections and restore either full data or specific databases/collections from it.
3.	Restore a specific collection from a single database backup
4.	Restore certain databases and/or collections from a full backup
5.	Make a point-in time recovery for the specified databases/collections. Available for replica sets only.
6. Perform selective restore of databases and collections, with the option to include users and roles defined in the database in your selective backup. 


## Sharded collections

!!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

You can back up and restore sharded collections. During backup, `pbm-agents` on each shard save the documents for the specified databases/collections and the full oplog for the period of the backup process. A `pbm-agent` on the config server replica set saves router config documents from the `config` database required for restoring the selected namespaces.

During the restore, the reverse process occurs:

* A `pbm-agent` on each shard restores only the specified databases/collections and replays the oplog that relates only to the specified namespaces. The operations for other namespaces are ignored.
* On the config server replica set, the `pbm-agent` restores the router configuration only for the specified sharded collections. The router configuration for other databases, collections and chunks remains intact.

The restore for sharded time series collections is not supported.

Note that selective backups and restores operate only with data and router configuration. The cluster configuration and topology-related settings are ignored. Therefore, we recommended to restore the databases/collections on the same environment.

### Implementation specifics

During the selective restore, the primary shard for a database is set to the state it had during the backup. For example, the primary shard for the database "Staff" during backup was A. After you restore the  "Staff" database, the primary shard will be set to A even if you moved the primary from A to B before the restore. All non-sharded collections will be restored on A; however, they will not be deleted from B. You must take needed actions (cleanup or move the primary back to B) to maintain them. 

## Restore with users and roles

### Overview

!!! admonition "Version added: [2.5.0](../release-notes/2.5.0.md)"

You can restore the specified databases with users and roles that were created against them. This feature is useful for deployments where each user has an individual database and authenticates against it. In such a way, you can recover desired datasets to their state prior to data corruption or loss.

Consider these specifics of selective restore with users and roles:

* You can restore custom databases from a full backup. 
* Users and roles must be created in custom databases. For security considerations, users created in `admin`, `config` and `local` databases cannot be a part of a selective restore.
* If users and roles exist in a database during the restore, they will be overwritten from the backup.

To restore a specific namespace and include users and roles, run the following command:

```bash
pbm restore --ns="mydb.*" --with-users-and-roles <backup-name>
```

To dive deep into this topic, see the section [selective restore with users and roles](../usage/restore-selective.md#restore-with-users-and-roles).


## Restore a collection under a different name

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)"

You can restore a specific collection under a different name alongside the current collection up to a certain point in time. This is useful when you troubleshoot database issues and need to compare the data in both collections to identify what caused the database to misbehave. When you can see and edit the changes explicitly, you gain insight into your data and have confident control over it. As a result, your troubleshooting efforts significantly reduce.

In version 2.8.0, you can restore a single non-sharded collection in a replica set under a different name. The support for sharded and time series collections is planned for the future releases.

To see how it works, imagine the following use case:

You have noticed that your e-commerce app returns incorrect or incomplete results on orders. You remember that everything was working fine yesterday, so it's likely that recent changes to the database caused the issue. 

You have a backup and the oplog ranges that fully cover the previous day, 2024-11-15.

To find out the issue, you can restore the `orders` collection under a different name up to the specified time alongside the current `orders` collection and compare them. You decide to restore to 14:00.

```bash
pbm restore --time=2024-11-15T14:00:00 --ns-from=goods.orders --ns-to=goods.orders_prev
```

The `orders_prev` collection has the same data and indexes as the `orders` collection. It also has applied the same oplog operations as the `orders` collection allowing you to see exactly what has changed.

Let's say you discover that the `status` field now includes an extra `date` field. These changes went unnoticed, and the app's code was not updated to handle them, leading to incorrect results. Now that you've identified the issue you can take necessary actions to fix it.

[Known limitations](../features/known-limitations.md#selective-backups-and-restores){.md-button}
[Make a backup](../usage/backup-selective.md){ .md-button .md-button }
[Restore a backup](../usage/restore-selective.md){ .md-button .md-button }
# Selective backup and restore

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

You can back up and restore certain namespaces - databases or collections. For example, if your "Payments" collection in the "Staff" database was corrupted, you can restore only this collection from your full backup up to a specific point in time. Or, if your "Invoices" database contains sensitive data and must be backed up frequently, you can configure the backup of only this database. This way you work only with the desired subset of data without disrupting the operations of your whole cluster. 

You also drastically reduce time on backup / restore operations of the whole data set and save on storage consumption.

With the selective backup and restore functionality, you have the following options:

1.	Backup a single database or a specific collection and restore all data from it. 
2.	Restore a specific collection from a single database backup
3.	Restore certain databases and / or collections from a full backup
4.	Make a point-in time recovery for the specified databases / collections. Available for replica sets only.

!!! warning

    Multi-collection transactions are not supported. However, if you use them and attempt a selective restore, it may break [ACID](../reference/glossary.md#acid) because not all operations with this transaction are restored. PBM applies oplog events that relate only to the specified namespaces(s). Thus, from the transaction's point of view, the data consistency may be broken.

    For example, you have a transaction that involves collections A and B. When you restore collection A, PBM replays oplog events only for collection A and ignores those related to collection B. As a result, the state of collection B remains unchanged and is no longer consistent with collection A. 


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

## Restore a database with users and roles

!!! admonition "Version added: [2.5.0](../release-notes/2.5.0.md)"

You can restore the specified databases with users and roles that were created against them. This feature is useful for deployments where each user has an individual database and authenticates against it. In such a way, you can recover desired datasets to their state prior to data corruption or loss.

Consider these specifics of selective restore with users and roles:

* You can restore custom databases from a full backup. 
* Users and roles must be created in custom databases. For security considerations, users created in `admin`, `config` and `local` databases cannot be a part of a selective restore.
* If users and roles exist in a database during the restore, they will be overwritten from the backup.

[Restore data with users and roles](../usage/restore.md#__tabbed_2_3){.md-button}

## Known limitations of selective backups and restores

1. Only **logical** backups and restores are supported.
2. Selective backups and restores are supported in sharded clusters for non-sharded collections starting with version 2.0.3. Sharded collections are supported starting with version 2.1.0. 
3. Sharded time series collections are not supported.
4. Multiple namespaces are not yet supported for selective backups. However, you can specify several namespaces for the restore (e.g., restore several collections of a database).
5. Multi-collection transactions are not yet supported for selective restore.
6. System collections in ``admin``, ``config``, and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
7. Selective point-in-time recovery is not supported for sharded clusters.

[Make a backup](../usage/start-backup.md){ .md-button .md-button }
[Restore a backup](../usage/restore.md){ .md-button .md-button }

# Selective backup and restore

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

!!! important

    Selective backup and restore is the [technical preview feature](../reference/glossary.md#technical-preview-feature).

You can back up and restore certain namespaces - databases or collections. For example, if your "Payments" collection in the "Staff" database was corrupted, you can restore only this collection from your full backup up to a specific point in time. Or, if your "Invoices" database contains sensitive data and must be backed up frequently, you can configure the backup of only this database. This way you work only with the desired subset of data without disrupting the operations of your whole cluster. 

You also drastically reduce time on backup / restore operations of the whole data set and save on storage consumption.

With the selective backup and restore functionality, you have the following options:

1.	Backup a single database or a specific collection and restore all data from it. 
2.	Restore a specific collection from a single database backup
3.	Restore certain databases and / or collections from a full backup
4.	Make a point-in time recovery for the specified databases / collections.


## Known limitations of selective backups and restores

1. Only **logical** backups and restores are supported.
2. Selective backups and restores are supported in sharded clusters for unsharded collections starting with version 2.0.3. Sharded collections are not yet supported.
3. Multiple namespaces are not yet supported for selective backups. However, you can specify several namespaces for the restore (e.g., restore several collections of a database).
4. Multi-collection transactions are not yet supported for selective restore.
5. System collections in ``admin``, ``config``, and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
6.	Point-in-time recovery slicing requires a full backup because it serves as the base for point-in-time recovery. Any selective backup will be ignored.

[Make a backup](../usage/start-backup.md){ .md-button .md-button }
[Restore a backup](../usage/restore.md){ .md-button .md-button }

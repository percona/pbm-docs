# Selective backup and restore

!!! admonition "Version added: 2.0.0"

!!! important

    Selective backup and restore is the technical preview feature [^1]

You can back up and restore certain namespaces - databases or collections. For example, if your "Payments" collection in the "Staff" database was corrupted, you can restore only this collection from your full backup up to a specific point in time. Or, if your "Invoices" database contains sensitive data and must be backed up frequently, you can configure the backup of only this database. This way you work only with the desired subset of data without disrupting the operations of your whole cluster. 

You also drastically reduce time on backup / restore operations of the whole data set and save on storage consumption.

With the selective backup and restore functionality you have the following options:

1.	Backup a single database or a specific collection and restore all data from it. 
2.	Restore a specific collection from a single database backup
3.	Restore certain databases and / or collections from a full backup
4.	Make a point-in time recovery for the specified databases / collections.

## Selective backup 

To make a selective backup,  run the `pbm backup` command and provide the value for the `--ns` flag in the format `<database.collection>`. The `--ns` flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

```sh
pbm backup –ns=staff.Payments
```

To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

```sh
pbm backup –ns=Invoices.*
```

During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is now the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 

## View information about a selective backup 

Selective backups are marked as ``selective`` in the ``pbm list`` and ``pbm status`` outputs:

```sh
pbm list
```

Output:

```
Backup snapshots:
  2022-08-17T10:03:29Z <logical> [restore_to_time: 2022-08-17T10:03:34Z]
  2022-08-17T10:49:03Z <logical, selective> [restore_to_time: 2022-08-17T10:49:08Z]
```

To view a detailed information about a backup, run the following command:

```sh
pbm describe-backup <backup-name>
```

The output provides the backup name, type, status, size, namespaces and the information about the cluster topology it was taken in:

Output:

```
name: "2022-08-17T10:49:03Z"
type: logical
last_write_ts: 1662039300,2
last_transition_ts: "1662039304"
namespaces:
- Invoices.*
mongodb_version: 5.0.10-9
pbm_version: 2.0.0
status: done
size: 10234670
error: ""
replsets:
- name: rs1
  status: done
  iscs: false
  last_write_ts: 1662039300,2
  last_transition_ts: "1662039304"
  error: ""
```

## Selective restore 

To restore a specific database or a collection, run the ``pbm restore`` command in the format:

```sh
pbm restore <backup_name> --ns <database.collection>
```

During the restore, Percona Backup for MongoDB retrieves the file for the specified database / collection and restores it.  

## Point-in-time recovery 

To start Point-in-time recovery oplog slicing, a full backup snapshot is required as it serves as the base for any restore. 

To restore the desired database or a collection to a point in time, run the ``pbm restore`` command as follows:

```sh
pbm restore --base-snapshot <backup_name> --time <timestamp> \
--ns <db.collection>
```

You can specify the selective backup as the base snapshot for the Point-in-time restore. In this case, Percona Backup for MongoDB restores only the namespace(s) included in this backup to the specified time.

Alternatively, you can use a full backup snapshot and restore the desired namespaces (databases or collections) up to the specific time from it. Specify them as the comma-separated list for the `pbm restore` command.

When point-in-time recovery is started, Percona Backup for MongoDB uses the provided base snapshot, restores the specified namespace(s) and replays oplog on top of it up to the specified time. If no base snapshot is provided, Percona Backup for MongoDB uses the most recent full backup snapshot.

## Known limitations of selective backups and restores

1. Only logical backups and restores are supported
2. Sharding is not supported.
3. Multiple namespaces are not yet supported for selective backups. Though you can specify several namespaces for the restore (e.g., restore several collections of a database).
4.	System collections in ``admin``, ``config`` and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
5.	Point-in-time recovery slicing requires a full backup because it serves as the base for point-in-time recovery. Any selective backup will be ignored.

[^1]: Tech Preview Features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.


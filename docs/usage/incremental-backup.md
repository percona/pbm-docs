# Incremental physical backups

!!! admonition "Version added: 2.3.0"

When owners of large datasets need to backup data frequently, making full physical backups every time is costly in terms of storage space. To optimize backup schemes and reduce storage costs, use incremental physical backups, available in Percona Backup for MongoDB starting with version 2.0.3. During incremental backups, Percona Backup for MongoDB saves only the data that was changed after the previous backup was taken. This results in faster backup / restore performance. Since incremental backups are smaller in size, you also save on storage costs.

## Considerations

* This is a [tech preview feature](../reference/glossary.md#technical-preview-feature). We recommend using it only for testing purposes. 

* Incremental backup implementation is based on the [`$backupCursor`](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html) aggregation stage that is available in only Percona Server for MongoDB. Therefore, you must be running Percona Server for MongoDB in your deployment to use incremental physical backups.

## Make incremental backups

To start incremental backups, first make a full incremental backup. It will serve as the base for subsequent incremental backups:

```bash 
pbm backup -type incremental --base
```

The `pbm-agent` starts tracking the incremental backup history to be able to calculate and save the difference in data blocks. After that you can run regular incremental backups:

```bash
pbm backup -type incremental
```

The incremental backup history looks like this:

```bash 
Snapshots:
    2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
    2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
    2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
    2022-11-24T14:45:53Z 220.13GB <physical> [restore_to_time: 2022-11-24T14:45:55Z]
```

## Restore from incremental backups

Restore flow from an incremental backup is the same as the restore from a full physical backup: specify the backup name for the `pbm restore` command:

```bash
pbm restore 2022-11-25T14:13:43Z
```

Percona Backup for MongoDB recognizes the backup type, finds the base incremental backup, restores the data from it and then restores the modified data from applicable incremental backups.

After the restore is complete, restart all `mongod` nodes, `pbm-agents` and resync the backup list from the storage

## Implementation specifics

Percona Backup for MongoDB tracks the backup history only on the node where the base incremental backup was taken. This means that subsequent incremental backups must always be run on that very node. 

If the node with the base incremental backup is down or unavailable, you must start the incremental backup chain anew on another node.

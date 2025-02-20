# Make a logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

During a *physical* backup, Percona Backup for MongoDB  copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage. 

To start a backup, run the following command:

```{.bash data-prompt="$"}
$ pbm backup --type=physical
```
     
!!! warning 

    During the period the backup cursor is open, database checkpoints can be created, but no checkpoints can be deleted. This may result in significant file growth.
    
Starting with [2.4.0](../release-notes/2.4.0.md), PBM doesn't stop [point-in-time recovery oplog slicing](../features/point-in-time-recovery.md#oplog-slicing), if it's enabled, but runs it in parallel. This ensures [point-in-time recovery](pitr-tutorial.md) to any timestamp if it takes too long (e.g. hours) to make a backup snapshot.

## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-physical.md){.md-button}
[Make a point-in-time recovery](pitr-physical.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


# Make a logical backup

--8<-- "prepare-backup.md"

## Procedure

!!! warning

    Sharded time series collections are not supported. If you use them in your deployment, you won't be able to make a backup. 
 

To make a backup, run the following command:

```{.bash data-prompt="$"}
$ pbm backup --type=logical
```
     
Logical backup is the default one so you can bypass the `--type` flag. 

During *logical* backups, Percona Backup for MongoDB copies the actual data to the backup storage.

Starting with version 2.0.0, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is now the default data format since it allows [selective restore](restore.md). Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB.



## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


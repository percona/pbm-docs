# Logical backups and restores

*Logical backup* is the copying of the actual database data. A `pbm-agent` connects to the database, retrieves the data, and writes it to the remote backup storage. 

Logical restore is the reverse process: The ``pbm-agent`` retrieves the backup data from the storage and inserts it on every primary node in the cluster. The remaining nodes receive the data during the replication process.

The following diagram shows the restore flow.

![image](../_images/pbm-restore-shard.png)


Logical backups allow for point-in-time recovery. 

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
| - Easy to operate with, using a single command <br> - Support for point-in-time recovery <br> - The backup size is smaller as it includes only the data | - Much slower than physical backup / restore <br> - Adds database overhead on reading and inserting the data| Sharded clusters and non-sharded replica sets | 

[Make a backup](../usage/start-backup.md){ .md-button .md-button }
[Restore a backup](../usage/restore.md){ .md-button .md-button }



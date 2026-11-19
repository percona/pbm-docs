> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Logical backups and restores

*Logical backup* is the copying of the actual database data. A `pbm-agent` connects to the database, retrieves the data, and writes it to the remote backup storage.

Logical restore is the reverse process: The `pbm-agent` retrieves the backup data from the storage and inserts it on every primary node in the cluster. The remaining nodes receive the data during the replication process.

Warning

Sharded time series collections are not supported.

During logical restores, oplog replay bypasses document validation to prevent validation errors from interrupting the restore. If you backed up time series collections during sustained writes, see [Restore time series collections](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore.html#restore-time-series-collections). It also explains how to prepare collections that have expiration configured.

The following diagram shows the restore flow.

| Advantages                                                                                                                                    | Disadvantages                                                                                           |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| - Easy to operate with, using a single command - Support for point-in-time recovery - The backup size is smaller as it includes only the data | - Much slower than physical backup / restore - Adds database overhead on reading and inserting the data |

[Make a backup](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/start-backup.html) [Restore a backup](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore.html)

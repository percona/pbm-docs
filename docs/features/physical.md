# Physical backups and restores

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

## Availability and system requirements

*  Percona Server for MongoDB starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher. 
* WiredTiger is used as the storage engine in Percona Server for MongoDB, since physical backups heavily rely on the WiredTiger [`$backupCursor`](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html) functionality.

!!! admonition "See also"

    Percona Blog

    * [Physical Backup Support in Percona Backup for MongoDB](https://www.percona.com/blog/physical-backup-support-in-percona-backup-for-mongodb/)
    * [$backupCursorExtend in Percona Server for MongoDB](https://www.percona.com/blog/2021/06/07/experimental-feature-backupcursorextend-in-percona-server-for-mongodb/)

Physical backup is copying of physical files from the Percona Server for MongoDB `dbPath` data directory to the remote backup storage. These files include data files, journal, index files, etc. Starting with version 2.0.0, Percona Backup for MongoDB also copies the WiredTiger storage options to the backup’s metadata. 

Physical restore is the reverse process: `pbm-agents` shut down the `mongod` nodes, clean up the `dbPath` data directory and copy the physical files from the storage to it. 

The following diagram shows the physical restore flow:

![image](../_images/pbm-phys-restore-shard.png)

During the restore, the ``pbm-agents`` temporarily start the ``mongod`` nodes using the the WiredTiger storage options retrieved from the backup’s metadata. The logs for these starts are saved to the ``pbm.restore.log`` file inside the ``dbPath``. Upon successful restore this file is deleted. However, it remains for debugging if the restore failed. 

During physical backups and restores, ``pbm-agents`` don’t export / import data from / to the database. This significantly reduces the backup / restore time compared to logical ones and is the recommended backup method for big (multi-terabyte) databases.

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
|- Faster backup and restore speed <br> - Recommended for big, multi-terabyte datasets <br> - No database overhead | - The backup size is bigger than for logical backups due to data fragmentation extra cost of keeping data and indexes in appropriate data structures <br> - Extra manual operations are required after the restore <br> - Point in time recovery requires manual operations | Sharded clusters and non-sharded replica sets |

[Make a backup](../usage/start-backup.md){ .md-button .md-button }
[Make a restore](../usage/restore.md){ .md-button .md-button }

## Physical restores with data-at-rest encryption

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

    You can backup and restore the encrypted data at rest. Thereby you ensure data safety and can also comply with security requirements such as GDPR, HIPAA, PCI DSS, or PHI.

This is how it works: 

During a backup, Percona Backup for MongoDB stores the encryption settings in the backup metadata. This allows you to verify them using the `pbm describe-backup` command. Note that the encryption key is not stored nor shown.

!!! important

    Make sure that you know what encryption key was used and store it as this key is required for the restore.

To restore the encrypted data from the backup, do the following:

1. Put the encryption key / specify the path to the key on at least one node of every replica set. 

2. Configure data-at-rest encryption in your destination cluster or replica set. 

During the restore, Percona Backup for MongoDB restores the data on the node where the encryption key matches the one with which the backed up data was encrypted. The other nodes are not restored, so the restore has the "partially done" status. You can start this node and initiate the replica set. The remaining nodes receive the data as the result of the initial sync from the restored node. 

Alternatively, you can place the encryption key to all nodes of the replica set. Then the restore is successful and complete on all nodes. This approach is faster and may suit for large data sets (terabytes of data). However, we recommend to rotate the encryption keys afterwards. Note, that key rotation is not available after the restore [for data-at-rest encryption with HashiCorp Vault key server](https://docs.percona.com/percona-server-for-mongodb/6.0/vault.html#vault). In this case, consider using the scenario with partially done restore.


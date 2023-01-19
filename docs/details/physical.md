# Physical backups and restores

!!! admonition "Version added: 1.7.0" 

Physical backup is copying of physical files from the Percona Server for MongoDB `dbPath` data directory to the remote backup storage. These files include data files, journal, index files, etc. Starting with version 2.0.0, Percona Backup for MongoDB also copies the WiredTiger storage options to the backup’s metadata. 

Physical restore is the reverse process: `pbm-agents` shut down the `mongod` nodes, clean up the `dbPath` data directory and copy the physical files from the storage to it. During this process, the ``pbm-agents`` temporarily start the ``mongod`` nodes using the the WiredTiger storage options retrieved from the backup’s metadata. The logs for these starts are saved to the ``pbm.restore.log`` file inside the ``dbPath``. Upon successful restore this file is deleted. However, it remains for debugging if the restore failed. 

During physical backups and restores, ``pbm-agents`` don’t export / import data from / to the database. This significantly reduces the backup / restore time compared to logical ones and is the recommended backup method for big (multi-terabyte) databases.

Physical backups and restores are available for Percona Server for MongoDB starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher. Since physical backups heavily rely on the WiredTiger `$backupCursor` functionality, they are available only for WiredTiger storage engine.

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
|- Faster backup and restore speed <br> - Recommended for big, multi-terabyte datasets <br> - No database overhead | - The backup size is bigger than for logical backups due to data fragmentation extra cost of keeping data and indexes in appropriate data structures <br> - Extra manual operations are required after the restore <br> - Point in time recovery requires manual operations | Sharded clusters and non-sharded replica sets |

!!! admonition "See also"

    Percona Blog

    * [Physical Backup Support in Percona Backup for MongoDB](https://www.percona.com/blog/physical-backup-support-in-percona-backup-for-mongodb/)
    * [$backupCursorExtend in Percona Server for MongoDB](https://www.percona.com/blog/2021/06/07/experimental-feature-backupcursorextend-in-percona-server-for-mongodb/)


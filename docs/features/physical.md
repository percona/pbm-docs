# Physical backups and restores

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

??? admonition "Implementation history"

    The following table lists the changes in the implementation of physical backups and the versions that introduced those changes:

    |Version | Description |
    |--------|-------------|
    | [2.0.0](../release-notes/2.0.0.md)   | Physical backups and restores, physical restore with data-at-rest encryption |
    | [2.3.0](../release-notes/2.3.0.md)   | Physical backups in mixed deployments |
    | [2.1.10](../release-notes/2.1.10.md) | Physical restore with a fallback directory | 

## Availability and system requirements

*  Percona Server for MongoDB starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher. 
* WiredTiger is used as the storage engine in Percona Server for MongoDB, since physical backups heavily rely on the WiredTiger [`$backupCursor` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html) functionality.

!!! warning 

    During the period the backup cursor is open, database checkpoints can be created, but no checkpoints can be deleted. This may result in significant file growth.
    
!!! admonition "See also"

    Percona Blog

    * [Physical Backup Support in Percona Backup for MongoDB :octicons-link-external-16:](https://www.percona.com/blog/physical-backup-support-in-percona-backup-for-mongodb/)
    * [$backupCursorExtend in Percona Server for MongoDB :octicons-link-external-16:](https://www.percona.com/blog/2021/06/07/experimental-feature-backupcursorextend-in-percona-server-for-mongodb/)

Physical backup is copying of physical files from the Percona Server for MongoDB `dbPath` data directory to the remote backup storage. These files include data files, journal, index files, etc. Starting with version 2.0.0, Percona Backup for MongoDB also copies the WiredTiger storage options to the backup's metadata. 

Physical restore is the reverse process: `pbm-agents` shut down the `mongod` nodes, clean up the `dbPath` data directory and copy the physical files from the storage to it. 

The following diagram shows the physical restore flow:

![image](../_images/pbm-phys-restore-shard.png)

During the restore, the ``pbm-agents`` temporarily start the ``mongod`` nodes using the WiredTiger storage options retrieved from the backup's metadata. The logs for these starts are saved to the ``pbm.restore.log`` file inside the ``dbPath``. Upon successful restore, this file is deleted. However, it remains for debugging if the restore were to fail. 

During physical backups and restores, ``pbm-agents`` don't export / import data from / to the database. This significantly reduces the backup / restore time compared to logical ones and is the recommended backup method for big (multi-terabyte) databases.

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
|- Faster backup and restore speed <br> - Recommended for big, multi-terabyte datasets <br> - No database overhead | - The backup size is bigger than for logical backups due to data fragmentation extra cost of keeping data and indexes in appropriate data structures <br> - Extra manual operations are required after the restore <br> - Point-in-time recovery requires manual operations | Sharded clusters and non-sharded replica sets |

[Make a backup](../usage/backup-physical.md){ .md-button }
[Restore a backup](../usage/restore-physical.md){ .md-button }

## Physical backups in mixed deployments

!!! admonition "Version added: [2.3.0](../release-notes/2.3.0.md)"

You may run both MongoDB Community / Enterprise Edition nodes and Percona Server for MongoDB (PSMDB) nodes in your environment, for example, when migrating to or evaluating PSMDB. 

You can make a physical, incremental or a snapshot-based backup in such a mixed deployment using PBM. This saves you from having to reconfigure your deployment for a backup, and keeps both your migration and backup strategies intact.

Physical, incremental and snapshot-based backups are only possible from PSMDB nodes since their implementation is based on the [`$backupCursorExtend` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html) functionality. When it's time to make a backup, PBM searches the PSMDB node and makes a backup from it. The PSMDB node must not be an arbiter nor a delayed node. 

If more than 2 nodes are suitable for a backup, PBM selects the one with a higher [priority](../usage/backup-priority.md). Note that if you override a priority for at least one node, PBM assigns priority `1.0` for the remaining nodes and uses the new priority list . 

Consider the following flow for [incremental backups](incremental-backup.md):
By default, PBM picks the node from where it made the incremental base backup when it makes subsequent backups. PBM assigns priority `3.0` to this node ensuring that it is the first in the list. If you change the node priority, make a new incremental base backup to ensure data continuity.

The physical restore in mixed deployments has no restrictions except the versions in backup and in the source cluster must match.

## Physical restores with data-at-rest encryption

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

You can back up and restore the data encrypted at rest. Thereby you ensure data safety and can also comply with security requirements such as GDPR, HIPAA, PCI DSS, or PHI.

This is how it works: 

During a backup, Percona Backup for MongoDB stores the encryption settings in the backup metadata. This allows you to verify them using the [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup) command. Note that the encryption key is not stored nor shown.

!!! important

    Make sure that you know what master encryption key was used and store it, as this key is required for the restore.

Starting with [Percona Server for MongoDB version 4.4.19-19 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.19-19.html), [5.0.15-13 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.15-13.html), [6.0.5-4 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.5-4.html) and higher, the master key rotation for data-at-rest encrypted with HashiCorp Vault has been improved to use the same secret key path on every server in your entire deployment. For the restore with earlier versions of Percona Server for MongoDB and PBM 2.0.5 and earlier, see the [Restore for Percona Server for MongoDB **before** 4.4.19-19, 5.0.15-13, 6.0.5-4 using HashiCorpVault](#restore-for-percona-server-for-mongodb-before-4419-19-5015-13-605-4-using-hashicorpvault) section.

To restore the encrypted data from the backup, configure data-at-rest encryption settings on all nodes of your destination cluster or replica set to match the settings of the target cluster where you made the backup

During the restore, Percona Backup for MongoDB restores the data all nodes using the same master key. To meet the security policy requirements in your organization, we recommend to rotate the master encryption keys afterwards. 

To learn more about master key rotation, refer to the following documentation:

* [Master key rotation in HashiCorp Vault server :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/vault.html#key-rotation)
* [KMIP master key rotation :octicons-link-external-16:](https://www.mongodb.com/docs/manual/tutorial/rotate-encryption-key/#kmip-master-key-rotation)

### Restore for Percona Server for MongoDB **before** 4.4.19-19, 5.0.15-13, 6.0.5-4 using HashiCorpVault

In Percona Server for MongoDB version **before** 4.4.19-19, 5.0.15-13, 6.0.5-4 with the Vault server used for data-at-rest encryption, the master key rotation with the same key used for 2+ nodes is not supported. If you run these versions of Percona Server for MongoDB and PBM before 2.1.0, consider using the scenario where PBM restores the data on one node of every replica set. The remaining nodes receive the data during the initial sync. 

Here's how it works:

Configure data-at-rest encryption on one node of every shard in your destination cluster or a replica set.

During the restore, Percona Backup for MongoDB restores the data on the node where the encryption key matches the one with which the backed up data was encrypted. The other nodes are not restored, so the restore has the "partially done" status. You can start this node and initiate the replica set. The remaining nodes receive the data as the result of the initial sync from the restored node.  

## Physical restores with a fallback directory

!!! admonition "Version added: [2.1.10](../release-notes/2.1.10.md)

During physical restore, PBM stops `mongod` instances, cleans up `dataDir`, uploads backup files from the storage and restarts `mongod` several times to apply the oplog and finalize the restore. This process is error-prone and may result in an unhealthy cluster or a replica set due to issues such as corrupt backup files, network connectivity issues, failures to restart `mongod`, and the like. The cluster may then become not operational and `pbm-agents` cannot connect to their `mongod` instances. 

To prevent this nasty situation, you can configure a fallback directory and revert the cluster to its original state if errors occur during a physical restore.  PBM copies the `dataDir` contents to the fallback directory at the restore start. Then the restore flows as usual.

If the restore is successful, PBM deletes the fallback directory and its contents. 

If PBM detects that the cluster is in error state, it automatically triggers the fallback procedure. PBM cleans up the uploaded backup files from the `dataDir` and moves the files from a fallback directory there. This way a cluster returns to the state before the restore and is operational. You can then retry the restore, try another backup or maintain it another way.

Note that this functionality comes with a tradeoff: you must have disk space at least twice the size of the `dataDir` to copy its contents in the fallback directory. For this reason, fallback directory usage is disabled by default. 

### Configuration

To configure restores with a fallback directory, use either the PBM configuration file or the command line:

=== "Configuration file"

    Specify the following option in the PBM configuration file:

    ```yaml
    restore:
      fallbackEnabled: true
    ```
    
    A restore can succeed on most nodes, but it might fail on a few, resulting in a "partlyDone" status. You can configure PBM how to proceed with such partial restores:

    ```yaml
    restore:
      allowPartialRestore: true
    ``` 
       
    If you allow them (default value), PBM finalizes the restore. Once the cluster is up and running, the failed node receives the necessary data from other members through an initial sync.

    If you deny partial restores, PBM treats a cluster as unhealthy and falls it back to the original state. In this case you must have the `restore.fallbackEnabled` option set to `true`
    
=== "Command line"

     You can start the restore with a fallback directory directly using the `--fallback` flag:

     ```{.bash data-prompt="$"}
     $ pbm restore --time <time> --fallback
     ```

     To allow partial restores, pass the `--allow-partial-restore` flag for the `pbm restore` command:

     ```{.bash data-prompt="$"}
     $ pbm restore --time <time> --allow-partial-restore
     ```

### Implementation specifics

1. The use of fallback directory is supported for both replica set and sharded cluster deployments
2. You must have the disk space at least twice the size of the `dataDir` to store its contents in the fallback directory during the restore.
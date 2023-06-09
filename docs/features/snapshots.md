# Snapshot-based physical backups

!!! admonition "Version added: [2.2.0](../release-notes/2.2.0.md)"

## Considerations 

1. This is a [technical preview feature](../reference/glossary.md#technical-preview-feature).
2. Supported only for physical backups
3. Available only if you run Percona Server for MongoDB in your environment  as PBM uses the [`$backupCursor and $backupCursorExtended aggregation stages`](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html). 

While a physical backup is a physical copy of your data directory, a snapshot is a point in time copy of your disk or a volume where the data files are stored. Restoring from snapshots is much faster and allows almost immediate access to data, while the database is unavailable during physical restore. Snapshot-based backups are especially useful for owners of large data sets with terabytes of data. Yet the snapshots don’t guarantee data consistency in sharded clusters.

This is where Percona Backup for MongoDB steps in. It provides the interface to make snapshot-based physical backups and restores and ensures data consistency. As a result, database owners benefit from increased performance and reduced downtime, and are sure that their data remains consistent.

The snapshot-based physical backup / restore flow consists of three distinct stages:

* Preparing the database — done by PBM
* Copying files — done by the user / client app
* Completing the backup / restore — done by PBM. 

This is the first stage of the snapshot-based backups where you can make them manually. Automated snapshot-based backups are planned for future releases.

## Make a backup

1. Refer to the [Before you start](../usage/start-backup.md#before-you-start) section and make sure that you have made all the preparation steps for the backup. 

2. To make a snapshot-based backup, run the [`pbm backup`](../reference/pbm-commands.md#pbm-backup) command with the type `external`:

    ```{.bash data-prompt="$"}
    $ pbm backup -t external --list-files
    ```    

    When executing the command, PBM does the following:    

    * opens the `$backupCursor`
    * prepares the database for file copy
    * stores the backup metadata on the storage and adds it to the files to copy
    * prints the prompt similar to the following:    

       ```{.text .no-copy}
       Ready to copy data from:
       <node-list>
       ```    

    You also see the backup name and the list of files to copy from each node. 

3. At this stage, you can copy files to the storage / make a snapshot using the technology of your choice.

4. (Optional) To check the backup progress, run the [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup). The command output provides the backup state and what nodes are running backup.

5. After the file copy, run the following command to close the `$backupCursor` and complete the backup: 

    ```{.bash data-prompt="$"}
    $ pbm backup-finish <backup_name>
    ```

## Restore a backup

### Before you start:

1. Shut down all `mongos` nodes.
2. Stop the arbiter nodes manually since there’s no `pbm-agent` on these nodes to do that automatically.

### Procedure

1. To make a restore, run the following command:

    ```{.bash data-prompt="$"}
    $ pbm restore --external 
    ```    

    Percona Backup for MongoDB prepares the database, provides the restore name and prompts you to copy the data:    

    ```{.text .no-copy}
    <example goes here>
    ``` 

2. Copy the data. To check what files to copy, run [`pbm describe-restore`](../reference/pbm-commands.md#pbm-descrbe-restore) command.

    !!! important 

        You must **copy the data on every node of a corresponding replica set in a cluster**. For example, copy files from `node1` of `rs1` in a backup onto `node1` of `rs1` in the target cluster and so on.

3. After you copied the files to the nodes, complete the restore with the following command:    

    ```{.bash data-prompt="$"}
    $ pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml>
    ```    

    At this stage, Percona Backup for MongoDB reads the metadata from the backup, maintains data consistency and starts the cluster / replica set. The database is restored to the timestamp specified in the `restore_to_time` of the metadata.

### Post-restore steps 

After the restore is complete, do the following:

1. Start all `mongod` nodes

2. Start all `pbm-agents`

3. Run the following command to resync the backup list with the storage:

    ```{.bash data-prompt="$"}
    $ pbm config --force-resync
    ``` 

4. Start the balancer and start `mongos` nodes.

5. Make a fresh backup to serve as the new base for future restores. 

# Snapshot-based physical backups

!!! admonition "Version added: [2.2.0](../release-notes/2.2.0.md)"

## Considerations 

1. Supported only for full backups
2. Available only if you run Percona Server for MongoDB in your environment as PBM uses the [`$backupCursor and $backupCursorExtended aggregation stages` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html). 

While a physical backup is a physical copy of your data directory, a snapshot is a point in time copy of your disk or a volume where the data files are stored. Restoring from snapshots is much faster and allows almost immediate access to data, however the database is unavailable during restore. Snapshot-based backups are especially useful for owners of large data sets with terabytes of data. Yet the snapshots don’t guarantee complete data consistency in sharded clusters.

This is where Percona Backup for MongoDB steps in. It provides the interface to make snapshot-based physical backups and restores and ensures data consistency. As a result, database owners benefit from increased performance and reduced downtime, and are sure that their data remains consistent.

The snapshot-based physical backup / restore flow consists of three distinct stages:

* Preparing the database — done by PBM
* Copying files — done by the user / client app
* Completing the backup / restore — done by PBM. 

This is the first stage of the snapshot-based backups where you can make them manually. Automated snapshot-based backups are planned for the future.

## Make a backup

1. Refer to the [Before you start](../usage/start-backup.md#before-you-start) section and make sure that you have made all the preparation steps for the backup. 

2. To make a snapshot-based backup, run the [`pbm backup`](../reference/pbm-commands.md#pbm-backup) command with the type `external`:

    ```{.bash data-prompt="$"}
    $ pbm backup -t external 
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

    You also see the backup name. 

3. (Optional) You can check the backup progress with the [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup). The command output provides the backup state and what nodes are running backup.

4. At this stage, you need to copy the `dataDir` contents of each node in the `<node-list>` to the storage / make a snapshot using the technology of your choice. 

5. After the copy/snapshot is complete, run the following command to close the `$backupCursor` and finish the backup: 

    ```{.bash data-prompt="$"}
    $ pbm backup-finish <backup_name>
    ```

## Restore a backup 

### Before you start:

1. Shut down all `mongos` nodes. If you have set up any automatic restart of the database, disable it.
2. Stop the arbiter nodes manually, since there’s no `pbm-agent` on these nodes to do that automatically.

### Restore from a backup made through PBM

The following procedure describes the restore process from backups [made through PBM](#make-a-backup). It is also possible to attempt restoring from snapshots made without PBM (this feature is experimental). See [Restore from a backup made outside PBM](#restore-from-a-backup-made-outside-pbm).

1. To perform a restore, run the following command:

    ```{.bash data-prompt="$"}
    $ pbm restore --external 
    ```    

    Percona Backup for MongoDB stops the database, cleans up data directories on all nodes, provides the restore name and prompts you to copy the data:    

    ```{.text .no-copy}
    Starting restore <restore_name> from '[external]'.................................................................................................................................Ready to copy data to the nodes data directory.
        After the copy is done, run: pbm restore-finish <restore_name> -c </path/to/pbm.conf.yaml>
        Check restore status with: pbm describe-restore <restore_name> -c </path/to/pbm.conf.yaml>
        No other pbm command is available while the restore is running!
    ``` 

2. Copy back the data. While a backup is made from a single node of a replica set, for the restore you must **copy the data to every node of the corresponding replica set in the cluster**. For example, given a backup of a 3-node replica set `rs1` where backup was taken from `node3`, copy back the data to all 3 nodes in `rs1`.

3. After you copied the files to the nodes, complete the restore with the following command:    

    ```{.bash data-prompt="$"}
    $ pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml>
    ```    

    At this stage, Percona Backup for MongoDB reads the metadata from the backup, prepares the data for the cluster / replica set start and ensures its consistency. The database is restored to the timestamp specified in the `restore_to_time` of the metadata.

    !!! note

        If you use a filesystem as the remote backup storage, both `pbm-agent` and `pbm` CLI must have the same permissions to it. To achieve this, run the `pbm restore-finish` command as the `mongod` user:

        ```{.bash data-prompt="$"}
        $ sudo -u mongod -s pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml> --mongodb-uri=MONGODB_URI
        ```

4. Optional. You can track the restore progress by running the [`pbm describe-restore`](../reference/pbm-commands.md#pbm-descrbe-restore) command.

#### Post-restore steps 

After the restore is complete, do the following:

1. Start all `mongod` nodes

2. Start all `pbm-agents`

3. Run the following command to resync the backup list with the storage:

    ```{.bash data-prompt="$"}
    $ pbm config --force-resync
    ``` 

4. Start the balancer and start `mongos` nodes.

5. Make a fresh backup to serve as the new base for future restores. 

### Restore from a backup made outside PBM

!!! Warning

    This feature is experimental.
    
!!! important

    For external backups made through PBM, PBM performs compatibility checks for the backup and the target cluster. If you restore a backup made without PBM, it cannot ensure that the backup was made properly and in a consistent manner. Therefore, the backup compatibility is your responsibility.

To restore an external backup made without PBM, you need to specify the following for the `pbm restore` command:

* a path to the configuration file of the `mongod` node on the source cluster from where the backup was made. This is the configuration file that PBM will use during the restore. It should contain the [storage options :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/configuration-options/#storage-options ) per replica set name, for example:

   ```yaml
   rs1:
       storage:
           directoryPerDB: true
   rs2:
       storage:
           directoryPerDB: true
   ```

   To restore data that was encrypted at rest, make sure data-at-rest encryption settings on the source and target clusters are the same. 

* a timestamp to restore to

To restore from a backup, do the following:

1. Start a restore

    ```{.bash data-prompt="$"}
    $ pbm restore --external -c </path/to/mongod.conf> --ts 
    ```

    If the path to the source cluster `mongod.conf` is undefined, PBM tries to retrieve the required configuration options from the `mongod.conf` of the target cluster.    

    If the timestamp to restore to is undefined, PBM looks into the actual data during the restore and defines the most recent common cluster time across all shards. PBM restores the database up to this time.

2. Next, copy the data files. Note that you must copy the data **to every data-bearing node of your cluster / replica set**.

3. Complete the restore by running:

    ```{.bash data-prompt="$"}
    $ pbm restore-finish <restore_name> -c </path/to/pbm.conf.yaml>
    ```    

    At this stage, Percona Backup for MongoDB prepares the data for the cluster / replica set start and ensures its consistency. 

    !!! note

        If you use a filesystem as the remote backup storage, both `pbm-agent` and `pbm` CLI must have the same permissions to it. To achieve this, run the `pbm restore-finish` command as the `mongod` user:

        ```{.bash data-prompt="$"}
        $ sudo -u mongod -s pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml> --mongodb-uri=MONGODB_URI
        ```

4. Don't forget to complete the [post-restore steps](#post-restore-steps).



# Make a logical backup

--8<-- "prepare-backup.md"

## Procedure

1. To make a snapshot-based backup, run the [`pbm backup`](../reference/pbm-commands.md#pbm-backup) command with the type `external`:

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

2. (Optional) You can check the backup progress with the [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup). The command output provides the backup state and what nodes are running backup.

3. At this stage, you need to copy the `dataDir` contents of each node in the `<node-list>` to the storage / make a snapshot using the technology of your choice. 

4. After the copy/snapshot is complete, run the following command to close the `$backupCursor` and finish the backup: 

    ```{.bash data-prompt="$"}
    $ pbm backup-finish <backup_name>
    ```



## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


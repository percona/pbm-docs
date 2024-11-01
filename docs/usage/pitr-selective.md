# Restore selected databases and collections

!!! important

    Supported only for replica sets.
    Available for logical backups.

1. Before you start:

    1. Read [known limitations for selective backups and restores](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).
    2. Check that you [have made a full backup](start-backup.md#make-a-backup) because it serves as the base for point-in-time recovery. Any selective backup is ignored.

2. To restore the desired database or a collection to a point in time, run the ``pbm restore`` command as follows:

    ```{.bash data-prompt="$"}
    $ pbm restore --base-snapshot <backup_name> --time <timestamp> \
    --ns <db.collection>
    ```

    You can specify the selective backup as the base snapshot for the Point-in-time restore. In this case, Percona Backup for MongoDB restores only the namespace(s) included in this backup to the specified time.    

    Alternatively, you can use a full backup snapshot and restore the desired namespaces (databases or collections) up to the specific time from it. Specify them as the comma-separated list for the `pbm restore` command.    

    When point-in-time recovery is started, Percona Backup for MongoDB uses the provided base snapshot, restores the specified namespace(s) and replays oplog on top of it up to the specified time. If no base snapshot is provided, Percona Backup for MongoDB uses the most recent full backup snapshot.
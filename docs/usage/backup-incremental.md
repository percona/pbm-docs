# Make incremental backups

--8<-- "prepare-backup.md"

## Procedure

1. To start incremental backups, first make a full incremental backup. It will serve as the base for subsequent incremental backups:

    ```{.bash data-prompt="$"} 
    $ pbm backup --type incremental --base
    ```

    The `pbm-agent` starts tracking the incremental backup history to be able to calculate and save the difference in data blocks. 

2. Run regular incremental backups:

    ```{.bash data-prompt="$"}
    $ pbm backup --type incremental
    ```

The incremental backup history looks like this:

??? example "Sample output"

    ```{.bash .no-copy} 
    Snapshots:
        2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
        2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
        2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
        2022-11-24T14:45:53Z 220.13GB <incremental, base> [restore_to_time: 2022-11-24T14:45:55Z]
    ```


## Next steps

[List backups](../usage/list-backup.md){.md-button}
[Make a restore](restore-incremental.md){.md-button}

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


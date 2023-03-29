# Delete backups

Use [`pbm delete-backup`](../reference/pbm-commands.md#pbm-delete-backup) to delete backup snapshots and [`pbm delete-pitr`](../reference/pbm-commands.md#pbm-delete-pitr) to delete point-in-time recovery oplog slices.

## Delete backup snapshots

### Considerations

1. You can only delete a backup that is not running (has the “done” or the “error” state). To check the backup state, run the [`pbm status`](../reference/pbm-commands.md#pbm-status) command.

2. To ensure oplog continuity for [point-in-time restore](pitr-tutorial.md), the `pbm delete-backup` command deletes any backup(s) except the following:

    * A backup snapshot that can serve as the base for any point-in-time recovery and has point-in-time recovery time ranges deriving from it. To delete such a backup, first [delete the oplog slices](#delete-oplog-slices) that are created  after the `restore-to time` value for this backup.

    * The most recent backup if point-in-time recovery is enabled and there are no oplog slices following this backup yet.

    To illustrate this, let’s take the following `pbm list` output:

    ```{.bash .no-copy}
    Backup snapshots:
      2022-10-05T14:13:50Z <logical> [restore_to_time: 2022-10-05T14:13:55Z]
      2022-10-06T14:52:42Z <logical> [restore_to_time: 2022-10-06T14:52:47Z]
      2022-10-07T14:57:17Z <logical> [restore_to_time: 2022-10-07T14:57:22Z]

    PITR <on>:
      2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z
    ```

    You can delete a backup `2022-10-06T14:52:42Z` since it has no point-in-time oplog slices. You cannot delete the following backups:

    - `2022-10-05T14:13:50Z` because it is the base for recovery to any point in time from the PITR time range `2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z`
    - `2022-10-07T14:57:17Z` because PITR is enabled and there are no oplog slices following it yet.

### Behavior

You can delete either a specified backup snapshot or all backup snapshots older than the specified time. Starting with version 2.0.0, you can also delete [selective backups](../features/selective-backup.md). 

=== "A specific backup"

     To delete a backup, specify the `<backup_name>` as an argument.

     ```{.bash data-prompt="$"}
     $ pbm delete-backup <backup_name>
     ```

=== "Backups older than the specified time"
    
    To delete backups that were created before the specified time, pass the `--older-than` flag to the `pbm delete-backup` command. Specify the timestamp as an argument for `pbm delete-backup` in the following format:

    * `%Y-%M-%DT%H:%M:%S` (for example, 2021-04-20T13:13:20Z) or
    * `%Y-%M-%D` (2021-04-20).

    #### Example

    View backups:

    ```{.bash data-prompt="$"}
    $ pbm list
    ```

    **Output**:

    ```{.text .no-copy}
    Backup snapshots:
      2021-04-20T20:55:42Z
      2021-04-20T23:47:34Z
      2021-04-20T23:53:20Z
      2021-04-21T02:16:33Z
    ```

    Delete backups created before the specified timestamp

    ```{.bash data-prompt="$"}
    pbm delete-backup -f --older-than 2021-04-21
    ```

    **Output**:

    ```{.text .no-copy}
    Backup snapshots:
      2021-04-21T02:16:33Z
    ```

By default, the ``pbm delete-backup`` command asks for your confirmation to proceed with the deletion. To bypass it, add the `-f` or
 `--force` flag.

 ```{.bash data-prompt="$"}
 $ pbm delete-backup --force 2021-04-20T13:45:59Z
 ```

!!! admonition ""

    For Percona Backup for MongoDB 1.5.0 and earlier versions, when you delete a backup, all oplog slices that relate to this backup are deleted too. For example, you delete a backup snapshot `2020-07-24T18:13:09` while there is another snapshot `2020-08-05T04:27:55` created after it.  The **pbm-agent** deletes only oplog slices that relate to `2020-07-24T18:13:09`.

    The same applies if you delete backups older than the specified time.

    Note that when point-in-time recovery is enabled, the most recent backup snapshot and oplog slices that relate to it are not deleted.

## Delete oplog slices

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

You can delete oplog slices saved before the specified time or all slices altogether. By deleting old and/or unnecessary slices, you can save storage space. 

### Behavior

To view oplog slices, run the [`pbm list`](../reference/pbm-commands.md#pbm-list) command. If you have [deleted the snapshot](#delete-backup-snapshots) and want to delete the respective oplog slices, run the `pbm list --unbacked` command to view them.

=== "Delete all oplog slices"

    Run the `pbm delete-pitr` and pass the `--all` flag:

    ```{.bash data-prompt="$"}
    $ pbm delete-pitr --all
    ```

=== "Earlier than the specified timestamp" 
    
    To delete slices that are made earlier than the specified time, run the `pbm delete-pitr` command with the `--older-than` flag and pass the timestamp for it. The timestamp must be in the following format:

    * `%Y-%M-%DT%H:%M:%S` (for example, 2021-07-20T10:01:18) or
    * `%Y-%M-%D` (2021-07-20).

    ```{.bash data-prompt="$"}
    $ pbm delete-pitr --older-than 2021-07-20T10:01:18
    ```

To enable [point-in-time recovery](pitr-tutorial.md) from the most recent backup snapshot, Percona Backup for MongoDB does not delete slices that were made after that snapshot. For example, if the most recent snapshot is `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` and you specify the timestamp `2021-07-20T07:05:44`, Percona Backup for MongoDB deletes only slices that were made before `2021-07-20T07:05:23Z`.

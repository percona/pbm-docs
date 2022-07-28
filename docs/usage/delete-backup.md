# Delete backups

Use the ``pbm delete-backup`` command to delete a specified backup or all backups older than the specified time.

The command deletes the backup regardless of the remote storage used:
either S3-compatible or a filesystem-type remote storage.

!!! note 

    You can only delete a backup that is not running (has the “done” or the “error” state).

As of version 1.4.0, ``pbm list`` shows only successfully completed backups. To check for backups with other states, run ``pbm status``.

To delete a backup, specify the `<backup_name>` as an argument.

```
pbm delete-backup 2020-12-20T13:45:59Z
```

By default, the ``pbm delete-backup`` command asks for your confirmation
to proceed with the deletion. To bypass it, add the `-f` or
`--force` flag.

```
pbm delete-backup --force 2020-04-20T13:45:59Z
```

To delete backups that were created before the specified time, pass the `--older-than` flag to the `pbm delete-backup` command. Specify the timestamp as an argument for `pbm delete-backup` in the following format:

* `%Y-%M-%DT%H:%M:%S` (for example, 2020-04-20T13:13:20Z) or
* `%Y-%M-%D` (2020-04-20).

## Example

View backups:

```sh
pbm list
```

**Output**:

```
Backup snapshots:
  2020-04-20T20:55:42Z
  2020-04-20T23:47:34Z
  2020-04-20T23:53:20Z
  2020-04-21T02:16:33Z
```

Delete backups created before the specified timestamp

```sh
pbm delete-backup -f --older-than 2020-04-21
```

**Output**:

```
Backup snapshots:
  2020-04-21T02:16:33Z
```
# Cancel a backup

You can cancel a running backup if, for example, you want to do
another maintenance of a server and don’t want to wait for the large backup to finish first.

To cancel the backup, use the `pbm cancel-backup` command.

```sh
pbm cancel-backup
Backup cancellation has started
```

After the command execution, the backup is marked as canceled in the [pbm status](../manage/troubleshooting.md#percona-backup-for-mongodb-status) output:

```sh
pbm status
```

**Output**:

`2020-04-30T18:05:26Z  Canceled at 2020-04-30T18:05:37Z`
# Cancel a backup

You can cancel a running backup if, for example, you want to do
another maintenance of a server and don’t want to wait for the large backup to finish first.

To cancel the backup, use the [`pbm cancel-backup`](../reference/pbm-commands.md#pbm-cancel-backup) command.

```{.bash data-prompt="$"}
$ pbm cancel-backup
Backup cancellation has started
```

After the command execution, the backup is marked as canceled in the [pbm status](../manage/status.md) output:

```{.bash data-prompt="$"}
$ pbm status
```

**Output**:

```{.bash .no-copy}
2020-04-30T18:05:26Z  Canceled at 2020-04-30T18:05:37Z
```
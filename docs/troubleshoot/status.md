# Percona Backup for MongoDB status

!!! admonition "Version added: [1.4.0](../release-notes/1.4.0.md)"

You can check the status of Percona Backup for MongoDB running in your MongoDB environment using the [`pbm status`](../reference/pbm-commands.md#pbm-status) command.

```{.bash data-prompt="$"}
$ pbm status
```

The output provides the information about:

* Your MongoDB deployment and `pbm-agents` running in it: to what `mongod` node each agent is connected, the Percona Backup for MongoDB version it runs and the agent’s state

* The currently running backups / restores, if any

* Backups stored in the remote backup storage: backup name, type, completion time, size and status (complete, canceled, failed)

* [Point-in-time recovery](../features/point-in-time-recovery.md) status (enabled or disabled)

* Valid time ranges for point-in-time recovery and the data size

This simplifies troubleshooting since the whole information is provided in one place.

??? example "Sample output"

    ```{.bash .no-copy}
    pbm status    

    Cluster:
    ========
    config:
      - config/localhost:27027 [P]: pbm-agent [v2.9.1] OK
      - config/localhost:27028 [S]: pbm-agent [v2.9.1] OK
      - config/localhost:27029 [S]: pbm-agent [v2.9.1] OK
    rs1:
      - rs1/localhost:27018 [P]: pbm-agent [v2.9.1] OK
      - rs1/localhost:27019 [S]: pbm-agent [v2.9.1] OK
      - rs1/localhost:27020 [S]: pbm-agent [v2.9.1] OK
    rs2:
      - rs2/localhost:28018 [P]: pbm-agent [v2.9.1] OK
      - rs2/localhost:28019 [S]: pbm-agent [v2.9.1] OK
      - rs2/localhost:28020 [S]: pbm-agent [v2.9.1] OK    

    PITR incremental backup:
    ========================
    Status [OFF]    

    Currently running:
    ==================
    (none)    

    Backups:
    ========
    S3 us-east-1 https://storage.googleapis.com/backup-test
       Snapshots:
        2025-05-30T08:05:33Z 321.33KB <incremental> success [restore_to_time: 2025-05-30T08:05:35Z]
        2025-05-30T08:02:13Z 513.85KB <incremental> success [restore_to_time: 2025-05-30T08:02:16Z]
        2025-05-30T07:31:19Z 251.44KB <incremental> success [restore_to_time: 2025-05-30T07:31:21Z]
        2025-05-30T07:30:54Z 308.60KB <incremental, base> success [restore_to_time: 2025-05-30T07:30:56Z]
        2025-05-30T07:30:24Z 0.00B <incremental, base> failed [ERROR: define source backup: not found] [2025-05-30T07:30:27Z]
        2025-05-30T07:27:33Z 332.20KB <physical> success [restore_to_time: 2025-05-30T07:27:35Z]
        2025-05-30T07:26:13Z 238.54KB <physical> success [restore_to_time: 2025-05-30T07:26:15Z]
      PITR chunks [1.10MB]:
        2025-05-30T07:26:16Z - 2025-05-30T07:56:34Z
    ```

## `pbm-agent` logs

!!! admonition "Version added: [1.4.0](../release-notes/1.4.0.md)"

To troubleshoot issues with specific events or node(s), use the [`pbm logs`](../reference/pbm-commands.md#pbm-logs) command.  It provides logs of all `pbm-agent` processes in your environment. 

`pbm logs` has the set of filters to refine logs for specific events like `backup`, `restore`, `pitr` or for a specific node, and to manage log verbosity level. For example, to view logs about a specific backup with the Debug verbosity level, run the `pbm logs` command as follows:

```{.bash data-prompt="$"}
$ pbm logs --severity=D --event=backup/2020-10-15T17:42:54Z
```

To learn more about available filters and usage examples, refer to [Viewing backup logs](../usage/logs.md).

## Backup progress tracking

If you have a large logical backup, you can track the backup progress in the logs of the `pbm-agent` that makes it. A line is appended every minute showing bytes copied vs. total size for the current collection.

Start a backup:

```{.bash data-prompt="$"}
$ pbm backup
```

Check backup progress:

1. Check what `pbm-agent` makes the backup:

    ```{.bash data-prompt="$"}
    pbm logs
    ```

2. Connect to the `mongod` server where the `pbm-agent` is running and check its logs

    ```{.bash data-prompt="$"}
    $ journalctl -u pbm-agent.service
    ```

    ??? example "Sample output"

        ``` {.bash .no-copy}
        2020/05/06 21:31:12 Backup 2020-05-06T18:31:12Z started on node rs2/localhost:28018
        2020-05-06T21:31:14.797+0300 writing admin.system.users to archive on stdout
        2020-05-06T21:31:14.799+0300 done dumping admin.system.users (2 documents)
        2020-05-06T21:31:14.800+0300 writing admin.system.roles to archive on stdout
        2020-05-06T21:31:14.807+0300 done dumping admin.system.roles (1 document)
        2020-05-06T21:31:14.807+0300 writing admin.system.version to archive on stdout
        2020-05-06T21:31:14.815+0300 done dumping admin.system.version (3 documents)
        2020-05-06T21:31:14.816+0300 writing test.testt to archive on stdout
        2020-05-06T21:31:14.829+0300 writing test.testt2 to archive on stdout
        2020-05-06T21:31:14.829+0300 writing config.cache.chunks.config.system.sessions to archive on stdout
        2020-05-06T21:31:14.832+0300 done dumping config.cache.chunks.config.system.sessions (1 document)
        2020-05-06T21:31:14.834+0300 writing config.cache.collections to archive on stdout
        2020-05-06T21:31:14.835+0300 done dumping config.cache.collections (1 document)
        2020/05/06 21:31:24 [##......................]   test.testt  130841/1073901  (12.2%)
        2020/05/06 21:31:24 [##########..............]  test.testt2   131370/300000  (43.8%)
        2020/05/06 21:31:24
        2020/05/06 21:31:34 [#####...................]   test.testt  249603/1073901  (23.2%)
        2020/05/06 21:31:34 [###################.....]  test.testt2   249603/300000  (83.2%)
        2020/05/06 21:31:34
        2020/05/06 21:31:37 [########################]  test.testt2  300000/300000  (100.0%)
        ```


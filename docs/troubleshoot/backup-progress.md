# Backup progress tracking

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


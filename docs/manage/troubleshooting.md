# Percona Backup for MongoDB diagnostics tools

Percona Backup for MongoDB provides diagnostics tools to operate data backups.

## pbm-speed-test

**pbm-speed-test** allows field-testing compression and backup upload speed of logical backups. You can use it:

* To check performance before starting a backup

* To find out what slows down the running backup

By default, **pbm-speed-test** operates with fake semi random data documents. To
run **pbm-speed-test** on a real collection, provide a valid MongoDB connection URI string for the `--mongodb-uri` flag.

Run **pbm-speed-test** for the full set of available commands.

### Compression test

```{.bash data-prompt="$"}
$ pbm-speed-test compression --compression=s2 --size-gb 10
```

Output:

```{.bash .no-copy}
Test started ....
10.00GB sent in 8s.
Avg upload rate = 1217.13MB/s.
```

**pbm-speed-test compression** uses the compression library from the config
file and sends a fake semi random data document (1 GB by default) to the
black hole storage. (Use the `pbm config` command to change the compression library).

To test compression on a real collection, pass the
`--sample-collection` flag with the `<my_db.my_collection>` value.

Run `pbm-speed-test compression --help` for the full set of supported flags:

```{.bash data-prompt="$"}
$ pbm-speed-test compression --help
usage: pbm-speed-test compression

Run compression test

Flags:
      --help                     Show context-sensitive help (also try
                                 --help-long and --help-man).
      --mongodb-uri=MONGODB-URI  MongoDB connection string
  -c, --sample-collection=SAMPLE-COLLECTION
                                 Set collection as the data source
  -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
      --compression=s2           Compression type
                                 <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
      --compression-level=COMPRESSION-LEVEL
                                 Compression level (specific to the compression type)
                                 <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
```

### Upload speed test

```{.bash data-prompt="$"}
$ pbm-speed-test storage --compression=s2
```

Output

```
Test started
1.00GB sent in 1s.
Avg upload rate = 1744.43MB/s.
```

`pbm-speed-test storage` sends the semi random data (1 GB by default) to the
remote storage defined in the config file. Pass the `--size-gb` flag to change the
data size.

To run the test with the real collection’s data instead of the semi random data,
pass the `--sample-collection` flag with the `<my_db.my_collection>` value.

Run `pbm-speed-test storage --help` for the full set of available flags:

```
$ pbm-speed-test storage --help
usage: pbm-speed-test storage

Run storage test

Flags:
      --help                     Show context-sensitive help (also try --help-long and --help-man).
      --mongodb-uri=MONGODB-URI  MongoDB connection string
  -c, --sample-collection=SAMPLE-COLLECTION
                                 Set collection as the data source
  -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
      --compression=s2           Compression type <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
      --compression-level=COMPRESSION-LEVEL
                                Compression level (specific to the compression type)
```

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

    Sample output:

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

## `pbm-agent` logs

!!! admonition "Version added: [1.4.0](../release-notes/1.4.0.md)"

To troubleshoot issues with specific events or node(s), use the [`pbm logs`](../reference/pbm-commands.md#pbm-logs) command.  It provides logs of all `pbm-agent` processes in your environment. 

`pbm logs` has the set of filters to refine logs for specific events like `backup`, `restore`, `pitr` or for a specific node, and to manage log verbosity level. For example, to view logs about a specific backup with the Debug verbosity level, run the `pbm logs` command as follows:

```{.bash data-prompt="$"}
$ pbm logs --severity=D --event=backup/2020-10-15T17:42:54Z
```

To learn more about available filters and usage examples, refer to [Viewing backup logs](../usage/logs.md).

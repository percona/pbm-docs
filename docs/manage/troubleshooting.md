# Troubleshooting Percona Backup for MongoDB

Percona Backup for MongoDB provides troubleshooting tools to operate data backups.

## pbm-speed-test

**pbm-speed-test** allows field-testing compression and backup upload speed of logical backups. You can use it:

* to check performance before starting a backup;

* to find out what slows down the running backup.

By default, **pbm-speed-test** operates with fake semi random data documents. To
run **pbm-speed-test** on a real collection, provide a valid MongoDB connection URI string for the `--mongodb-uri` flag.

Run **pbm-speed-test** for the full set of available commands.

### Compression test

```sh
$ pbm-speed-test compression --compression=s2 --size-gb 10
```

Output:

```
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

```
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

```sh
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

If you have a large backup you can track backup progress in `pbm-agent` logs. A line is appended every minute showing bytes copied vs. total size for the current collection.

Start a backup:

```sh
$ pbm backup
```

Check backup progress:

```sh
$ journalctl -u pbm-agent.service
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

## Percona Backup for MongoDB status

!!! admonition "Version added: 1.4.0"

You can check the status of Percona Backup for MongoDB running in your MongoDB environment using the [`pbm status`]() command.

```sh
$ pbm status
```

The output provides the information about:

* Your MongoDB deployment and `pbm-agents` running in it: to what `mongod` node each agent is connected, the Percona Backup for MongoDB version it runs and the agent’s state

* The currently running backups / restores, if any

* Backups stored in the remote backup storage: backup name, completion time, size and status (complete, canceled, failed)

* [Point-in-Time Recovery](../usage/point-in-time-recovery.md) status (enabled or disabled).

* Valid time ranges for point-in-time recovery and the data size

This simplifies troubleshooting since the whole information is provided in one place.

**Sample output**

```sh
pbm status

Cluster:
========
config:
  - config/localhost:27027: pbm-agent v1.3.2 OK
  - config/localhost:27028: pbm-agent v1.3.2 OK
  - config/localhost:27029: pbm-agent v1.3.2 OK
rs1:
  - rs1/localhost:27018: pbm-agent v1.3.2 OK
  - rs1/localhost:27019: pbm-agent v1.3.2 OK
  - rs1/localhost:27020: pbm-agent v1.3.2 OK
rs2:
  - rs2/localhost:28018: pbm-agent v1.3.2 OK
  - rs2/localhost:28019: pbm-agent v1.3.2 OK
  - rs2/localhost:28020: pbm-agent v1.3.2 OK

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
     2020-12-16T10:36:52Z 491.98KB [restore_to_time: 2020-12-16T10:37:13Z]
     2020-12-15T12:59:47Z 284.06KB [restore_to_time: 2020-12-15T13:00:08Z]
     2020-12-15T11:40:46Z 0.00B [canceled: 2020-12-15T11:41:07Z]
     2020-12-11T16:23:55Z 284.82KB [restore_to_time: 2020-12-11T16:24:16Z]
     2020-12-11T16:22:35Z 284.04KB [restore_to_time: 2020-12-11T16:22:56Z]
     2020-12-11T16:21:15Z 283.36KB [restore_to_time: 2020-12-11T16:21:36Z]
     2020-12-11T16:19:54Z 281.73KB [restore_to_time: 2020-12-11T16:20:15Z]
     2020-12-11T16:19:00Z 281.73KB [restore_to_time: 2020-12-11T16:19:21Z]
     2020-12-11T15:30:38Z 287.07KB [restore_to_time: 2020-12-11T15:30:59Z]
PITR chunks:
     2020-12-16T10:37:13 - 2020-12-16T10:43:26 44.17KB
```

## `pbm-agent` logs

!!! admonition "Version added: 1.4.0"

To troubleshoot issues with specific events or node(s), use the [`pbm logs`](../reference/pbm-commands.md#pbm-logs) command.  It provides logs of all `pbm-agent` processes in your environment. 

`pbm logs` has the set of filters to refine logs for specific events like `backup`, `restore`, `pitr` or for a specific node, and to manage log verbosity level. For example, to view logs about a specific backup with the Debug verbosity level, run the `pbm logs` command as follows:

```sh
$ pbm logs --severity=D --event=backup/2020-10-15T17:42:54Z
```

To learn more about available filters and usage examples, refer to [Viewing backup logs](../usage/logs.md).

# `pbm` commands

`pbm` CLI is the command line utility to control the backup system. This page describes `pbm` commands available in Percona Backup for MongoDB.

For how to get started with Percona Backup for MongoDB, see [Initial setup](../install/initial-setup.md).

## pbm backup

Creates a backup snapshot and saves it in the remote backup storage.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm backup [<flags>]
```

For more information about using `pbm backup`, see [Starting a backup](../usage/start-backup.md)

The command accepts the following flags:

| Flag           | Description                                           |
| -------------- | ----------------------------------------------------- |
| `-t`, `--type` | The type of backup. Supported values: physical, logical (default), incremental, [external](../features/snapshots.md). When not specified, Percona Backup for MongoDB makes a logical backup. |
| `--base`       | For incremental backups only. Set the backup as the base and start tracking the incremental backup history to calculate and save the difference in data blocks for subsequent incremental backups. |  
| `--compression`| Create a backup with compression. <br> Supported compression methods: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`. Default: `s2` <br> The `none` value means no compression is done during backup. |
| `--compression-level` | Configure the compression level from 0 to 10. The default value depends on the compression method used.  |
| `--num-parallel-collections`| Sets the number of collections to process in parallel during a specific logical backup. When undefined, `pbm-agent` processes the number of parallel collections defined for the `backup.numParallelCollections` configuration parameter. If that is undefined, the default number of collections to process in parallel is the half of the number of logical CPUs. Available starting with version 2.7.0.|
| `-o`, `--out=text`    | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |
| `--wait`       | Wait for the backup to finish. The flag blocks the shell session.|
| `--wait-time`  | The time to wait for PBM to report the status of the command execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.| 
| `-l`, `--list-files` | For external backups only. Shows the list of files per node to copy.|
| `--ns="database.collection"`| Makes a logical backup of the specified namespace - the database and collection(s). To back up all collections in the database, specify the value in the `--ns="database.*"` format. Starting with version 2.8.0, you can pass multiple namespaces as a comma-separated list for the backup. The format is `ns=db1.*,db2.coll2,db3.coll1,db3.collX` .|

??? "JSON output"

    ```json
    {
      "name": "<backup_name>",
      "storage": "<my-backup-dir>"
    }
    ```

## pbm backup-finish

Closes the `backupCursor` and finishes the external backup. Must be run after running `pbm backup -t external`. To learn more, refer to [API for snapshot-based physical backups](../features/snapshots.md).

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm backup-finish [backup-name] 
```

## pbm cancel-backup

Cancels a running backup. The backup is marked as canceled in the backup list.

The command accepts the following flags:

| Flag                | Description              | 
| ------------------- | ------------------------ |
| `-o`, `--out=text`  | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json`         |

??? "JSON output"

    ```json
    {
      "msg": "Backup cancellation has started"
    }
    ```

## pbm cleanup

Deletes outdated backup snapshots and point-in-time recovery oplog slices.

The command has the following syntax:

```{.bash data-prompt="$"}
pbm cleanup [<flags>]
```

The command accepts the following flags:

| Flag                     | Description               |
| ------------------------ | ------------------------- |
| `--older-than=TIMESTAMP` | Deletes backups older than date / time specified in the format:<br> - `%Y-%M-%DT%H:%M:%S` (e.g. 2020-04-20T13:13:20), <br> - `%Y-%M-%D` (e.g. 2020-04-20), <br> - `XXd` (e.g. 30d). Only days are supported|
| `-w`, `--wait`           | Wait for the cleanup to finish. The flag blocks the shell session|
| `--wait-time`  | The time to wait for PBM to report the status of the command execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.| 
| `-y`, `--yes`            | Cleans up the data storage without asking for a user's confirmation|
| `--dry-run`              | Checks for the old data to be deleted without deleting it. Allows to verify what data to delete| 


## pbm config

Sets, changes or lists Percona Backup for MongoDB configuration.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm config [<flags>] [<key>]
```

The command accepts the following flags:

| Flag               | Description                           |
| ------------------ | ------------------------------------- | 
| `--force-resync`   | Resync PBM's metadata (backup, point-in-time recovery chunks, restore) stored within PBM control collections with the data from the current storage. <br> Starting with version 2.10.0, PBM retrieves restore metadata from the storage only for the latest restore to improve resync performance. To retrieve the full restore history, also add the `--include-restores` flag. Resync for backup and point-in-time recovery chunks remains unchanged.|            
| `--list`           | List current settings                  |
| `--file=FILE`      | Upload the config information from a YAML file   |
| `--set=SET`        | Set a new config option value. Specify the option in the `<key.name=value>` format.                                    |
| `-o`, `--out=text` | Shows the output format as either plain text or a JSON object. Supported values: text, json                      |
| `-w`, `--wait`     | Wait for resync of the backup list with the storage to finish. You can only use this flag together with the `--force-resync` flag.|
| `--wait-time`  | The time to wait for PBM to report the status of the resync execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.| 
| `--include-restores`| Resync the full restore metadata history from the storage. Use this flag together with the `--force-resync` flag. Note that retrieving the full restore history may affect resync performance. Available starting with version 2.10.0. |


??? "PBM configuration output"

    ```json
    {
      "pitr": {
        "enabled": false,
        "oplogSpanMin": 0
      },
      "storage": {
        "type": "filesystem",
        "s3": {
          "region": "",
          "endpointUrl": "",
          "bucket": ""
        },
        "azure": {},
        "filesystem": {
          "path": "<my-backup-dir>"
        }
      },
      "restore": {
        "batchSize": 500,
        "numInsertionWorkers": 10
      },
      "backup": {}
    }
    ```

??? "Setting a config value"   

    ```json
    [
      {
        "key": "pitr.enabled",
        "value": "true"
      }
    ]
    ``` 

## pbm delete-backup

Deletes the specified backup snapshot or all backup snapshots that are older than the specified time with the option to filter by specific type. The command deletes backups that are not running regardless of the remote backup storage being used.

The following is the command syntax:

```{.bash data-prompt="$"}
$ pbm delete-backup [<flags>] [<name>]
```

The command accepts the following flags:

| Flag                     | Description             |
| ------------------------ | ----------------------- |
| `--older-than=TIMESTAMP` | Deletes backups older than date / time specified in the format:<br> - `%Y-%M-%DT%H:%M:%S` (e.g. 2023-04-20T13:13:20) or <br> - `%Y-%M-%D` (e.g. 2023-04-20)|
| `--type=TYPE`           | Deletes backups of the specified type. Must be used together with the `-older-than` flag. Available starting with version 2.4.0|
| `--force`                | Forcibly deletes backups without asking for user's confirmation. Deprecated. Use the `--yes` flag instead. |
| `-y`, `--yes`            | Deletes backups without asking for user's confirmation |
| `--dry-run`              | Prints the list of backup snapshots to be deleted without deleting them. You can use the flag to check what exactly will be deleted. Available starting with version 2.4.0. | 

## pbm delete-pitr

Deletes oplog slices produced for Point-in-Time Recovery.

The command has the following syntax:

```{.bash data-prompt="$"}
pbm delete-pitr [<flags>]
```

The command accepts the following flags:

| Flag                     | Description               |
| ------------------------ | ------------------------- |
| `-a`, `--all`            | Deletes all oplog slices. Deprecated. Use the `--older-than flag instead`  |
| `--older-than=TIMESTAMP` | Deletes oplog slices older than date / time specified in the format: <br> - `%Y-%M-%DT%H:%M:%S` (e.g. 2020-04-20T13:13:20) or <br> - `%Y-%M-%D` (e.g. 2020-04-20) <br><br> When you specify a timestamp, Percona Backup for MongoDB rounds it down to align with the completion time of the closest backup snapshot and deletes oplog slices that precede this time. Thus, extra slices remain. This is done to ensure oplog continuity. To illustrate, the PITR time range is `2021-08-11T11:16:21 - 2021-08-12T08:55:25` and backup snapshots are: <br><br> `2021-08-12T08:49:46Z 13.49MB [restore_to_time: 2021-08-12T08:50:06]` <br> `2021-08-11T11:36:17Z 7.37MB [restore_to_time: 2021-08-11T11:36:38]`<br> <br> Say you specify the timestamp `2021-08-11T19:16:21`. The closest backup is `2021-08-11T11:36:17Z 7.37KB [restore_to_time: 2021-08-11T11:36:38]`. PBM rounds down the timestamp to `2021-08-11T11:36:38` and deletes all slices that precede this time. As a result, your PITR time range is `2021-08-11T11:36:38 - 2021-08-12T09:00:25`. <br><br> **NOTE**: Percona Backup for MongoDB doesn’t delete the oplog slices that follow the most recent backup. This is done to ensure point in time recovery from that backup snapshot. For example, if the snapshot is `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` and you specify the timestamp `2021-07-20T07:05:45`, Percona Backup for MongoDB deletes only slices that were made before `2021-07-20T07:05:23Z`.|
| `--force`                | Forcibly deletes oplog slices without asking a user’s confirmation. Deprecated. Use the `-y`/`--yes` flag instead.  |
| `-o`, `--out=json`       | Shows the output as either the plain text (default) or a JSON object. Supported values: `text`, `json`.   |
| `--yes`                  | Deletes backups without asking for user's confirmation |
| `--dry-run`              | Prints the list of oplog slices to be deleted without deleting them. You can use the flag to check what exactly will be deleted. Available starting with version 2.4.0. | 
| `-w`, `--wait`          | Wait for the deletion operation to complete. |
| `--wait-time`  | The time to wait for PBM to report the status of the command execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.| 


## pbm describe-backup

Provides the detailed information about a backup:

- backup name
- type
- status
- namespaces - what was backed up during a selective backup
- size
- error message for failed backup
- last write timestamp 
- last write time - human-readable indication of the last write 
- last transition time - the timestamp when a backup changed its status
- cluster information: the replica set name, the backup status on this replica set, whether it is used as a config server replica set, last write timestamp
- replica set info: name, backup status, last write timestamp and last transition time, `mongod` security options, if encryption is configured.
- for snapshot-based backups, provides the list of files being copied
- for logical and selective backups, provides the list of collections included in the backup. Available with version 2.3.0.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm describe-backup [<backup-name>] [<flags>] 
```

| Flag                  | Description                           |
| --------------------- | ------------------------------------- |
| `-o`, `--out=text`    | Shows the status as either plain text or a JSON object. Supported values: `text`, `json`|
| `-l`, `--list-files`  | Shows the list of files being copied for snapshot-based backups |
| `--with-collections`  | Shows the collections included in the backup. For logical and selective backups only. Available with version 2.3.0.

### Output

The output document contains the following fields: 

??? admonition "JSON output"

    ```json
    {
      "name": "<backup_name>",
      "opid": "<string>",
      "type": "logical",
      "last_write_ts": Timestamp,
      "last_transition_ts": Timestamp,
      "last_write_time": "2022-09-30T14:02:49Z",
      "last_transition_time": "2022-09-30T14:02:54Z",
      "namespaces": [
        "flight.booking"
      ],
      "mongodb_version": "<version>",
      "fcv": "<version>",
      "pbm_version": "<version>",
      "status": "done",
      "size": 470805945,
      "size_h": "449.0 MiB",
      "replsets": [
        {
          "name": "<name>",
          "status": "done",
          "node": "example.mongodb.com:27017",
          "last_write_ts": Timestamp,
          "last_transition_ts": Timestamp,
          "last_write_time": "2022-09-30T14:02:49Z",
          "last_transition_time": "2022-09-30T14:02:53Z",
          "configsvr": true,
          "security": {}
        },
        {...},
        {...}
      ]
    }
    ```

| Field       | Description |
| ----------- | ----------- |
| `name`      | The backup name |
| `opid`      | A unique identifier of an operation |
| `type`      | The backup type. Supported values: logical, physical, incremental, external |
| `last_write_ts` | The timestamp of the last write |
| `last_transition_ts` | The timestamp when a backup changed its status |
| `last_write_time` | The human-readable indication of the last write |
| `last_transition_time` | The human-readable indication of the time when a backup changed its status|
| `namespaces` | The list of namespaces included in the backup. Available for selective backup |
| `mongodb_version` | The MongoDB version |
| `fcv` | The feature compatibility version |
| `pbm_version` | The Percona Backup for MongoDB version |
| `status` | The backup status. Supported values: running, dumpDone, done, copyReady, error, canceled |
| `size` | The backup size in bytes |
| `size_h` | The backup size in human-readable format |
| `error`  | The error message for a failed backup |
| `replsets` | The list of replica sets included in the backup. Each replica set has the following fields: <br> - `name` - the replica set name <br> - `status` - the backup status on this replica set <br> - `node` - the node name and port <br> - `last_write_ts` - the timestamp of the last write <br> - `last_transition_ts` - the timestamp when a backup changed its status <br> - `last_write_time` - the human-readable indication of the last write <br> - `last_transition_time` - the human-readable indication of the time when a backup changed its status <br> - `security` - the security options of the `mongod` process <br> - `configsvr` - indicates that this is a config server replica set |


## pbm describe-restore

Shows the detailed information about the restore:

* Restore name
* opID
* The name of the backup from which the database was restored
* Type
* Status
* Start and finish time of the restore
* Last transition time – the time when the restore process changed its status
* The name of every replica set, its restore status and the last transition time

For **physical backups only**, the following additional information is provided:

* The node name
* Restore status on the node
* Last transition time

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm describe-restore [<restore-timestamp>] [<flags>] 
```

The command accepts the following flags:

| Flag                     | Description             |
| ------------------------ | ----------------------- |
| `-c`, `--config=CONFIG`  | Only for **physical restores**. Points Percona Backup for MongoDB to a configuration file so it can read the restore status from the remote storage. For example, `pbm describe-restore -c /etc/pbm/conf.yaml <restore-name>`.|
| `-o`, `--out=TEXT`       | Shows the output as either the plain text (default) or a JSON object. Supported values: ``text``, ``json``.|

### Output

The output document contains the following fields:

??? admonition "Selective restore status"

    ```json
    {
     "name": "<restore_name>",
     "opid": "string",
     "backup": "<backup_name>",
     "type": "logical",
     "status": "done",
     "start": "Time"
     "finish": "Time"
     "namespaces": [
        "<database.*>"
     "last_transition_time": "Time"
     ]
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_time": "Time"
       },
       {
        "name": "rs0",
        "status": "done",
        "last_transition_time": "Time"
       },
       {
         "name": "cfg",
         "status": "done",
         "last_transition_time": "Time"
       }
     ],
    }
    ```

??? admonition "Physical restore status"

    ```json
    {
     "name": "<restore_name>",
     "opid": "string",
     "backup": "<backup_name>",
     "type": "physical",
     "status": "done",
     "start": "Time"
     "finish": "Time"
     "last_transition_time": "Time",
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_time": "Timestamp",
         "nodes": [
           {
             "name": "IP:port",
             "status": "done",
             "last_transition_time": "Timestamp"
           }
         ]
       }
     ],
    }
    ```

| Field       | Description |
| ----------- | ----------- |
| `name`      | The restore name |
| `opid`      | A unique identifier of an operation |
| `backup`    | The name of the backup from which the database was restored |
| `type`      | The restore type. Supported values: logical, physical|
| `status`    | The restore status. Supported values: running, copyReady, done, error |
| `start`     | The time when the restore started |
| `finish`    | The time when the restore finished. Available only for successful restores (with the status `done`) | 
| `error`     | The error message for a failed restore |
| `last_transition_time` | The human-readable indication of the time when the restore process changed its status |
| `namespaces` | The list of namespaces included in the restore. Available for selective restore |
| `replsets`  | The list of replica sets included in the restore. Each replica set has the following fields: <br> - `name` - the replica set name <br> - `status` - the restore status on this replica set <br> - `error` - the error message for failed restore <br> - `last_transition_time` - the human-readable indication of the time when the restore process changed its status <br> - `nodes` - the list of nodes included in the restore. |
| `replsets.nodes`     | The list of nodes included in the restore. Each node has the following fields: <br> - `name` - the node name and port <br> - `status` - the restore status on the node <br> - `error` - the error message for failed restore <br> - `last_transition_time` - the human-readable indication of the time when the restore process changed its status |

## pbm diagnostic

Generates the report with the detailed information about a specific backup or a restore. You can also use it for other commands. To learn more, refer to the [Diagnostics report](../troubleshoot/pbm-report.md).

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm diagnose --path path --name <backup-name> --opid <OPID>
```

The command accepts the following flags:

| Flag                | Description                      |
| ------------------- | -------------------------------- |
| `--path`            | The path where to save the report. If the directory doesn’t exist, PBM creates it during the report generation. Make sure that the user that runs PBM CLI has write access to the specified path |
| `--name`            | The name of the required backup or a restore |
| `--opid`            | The unique Operation ID of the specified command. You can retrieve it from the `pbm logs`, `pbm describe-backup` / `pbm describe-restore` output. |
| `--archive`         | Creates a .zip archive of the report in the specified path.|


## pbm help

Returns the help information about `pbm` commands.

## pbm list

Provides the list of backups. In versions 1.3.4 and earlier, the command lists all backups and their states. Backup states are the following:

* In progress - A backup is running
* Canceled - A backup was canceled
* Error - A backup was finished with an error
* No status means a backup is complete

As of version 1.4.0, only successfully completed backups are listed. To view currently information about a running or a failed backup, run [`pbm status`](#pbm-status).

When Point-in-Time Recovery is enabled, the `pbm list` also provides the list of valid time ranges for recovery and point-in-time recovery status.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm list [<flags>]
```

The command accepts the following flags:

| Flag                | Description                      |
| ------------------- | -------------------------------- |
| `--restore`         | Shows last N restores. Starting with version 2.0, the output shows restore names instead of backup names, as multiple restores can be done from a single backup.           |
| `--size=0`          | Shows last N backups.  It also provides the information whether the restore is a selective one.         |
| `-o`, `--out=text`  | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json`                 |
| `--unbacked`        | Shows Point-in-Time Recovery oplog slices that were saved without the base backup snapshot. Available starting with version 1.8.0.|
| `--replset-remapping` | Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|

??? "List of backups"

    ```json
    {
      "snapshots": [
          {
            "name": "<backup-name>",
            "status": "done",
            "restoreTo": Timestamp,
            "pbmVersion": "<version>",
            "type": "logical",
            "src": ""
          },
          {
            "name": "<backup-name>",
            "status": "done",
            "restoreTo": Timestamp,
            "pbmVersion": "<version>",
            "type": "logical",
            "src": "",
            "storage": "<storage-name>"
          }
      ],
      "pitr": {
        "on": false,
        "ranges": [
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp
            }
          },
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp
            },
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp (no base snapshot)
            }
          }
        ]
      }
    }
    ```

??? "Restore history"
 
    Full restore 

    ```json
     {
        "start": Timestamp,
        "status": "done",
        "type": "snapshot",
        "snapshot": "<backup_name>",
        "name": "<restore_name>"
      }
    ```

    Selective restore

    ```json
      {
        "start": Timestamp,
        "status": "done",
        "type": "snapshot",
        "snapshot": "<backup_name>",
        "name": "<restore_name>",
        "namespaces": [
          "<database.collection>"
        ]
      }
    ```

    Point-in-time restore

    ```json
      {
        "start": Timestamp,
        "status": "done",
        "type": "pitr",
        "snapshot": "<backup_name>",
        "point-in-time": Timestamp,
        "name": "<restore_name>"
      }
    ```

    Selective point-in-time restore

    ```json
    {
        "start": Timestamp,
        "status": "done",
        "type": "pitr",
        "snapshot": "<backup_name>",
        "point-in-time": Timestamp,
        "name": "<restore_name>",
        "namespaces": [
          "<database.collection>"
        ]
      }
    ]
    ```

## pbm logs

Shows log information from all `pbm-agent` processes.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm logs [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-t`, `--tail=20`       | Shows last N entries. By default, the output shows last 20 entries. <br> `0` means to show all log messages. |
| `-e`, `--event=EVENT`   | Shows logs filtered by a specified event. Supported events:<br> - backup<br> - restore <br> - resyncBcpList <br> - pitr <br> - pitrestore <br> - delete <br>  |
| `-o`, `--out=text`      | Shows log information as text (default) or in JSON format. <br> Supported values: `text`, `json` |
| `-n`, `--node=NODE`     | Shows logs for a specified node or a replica set.<br> Specify the node in the format `replset[/host:port]` |
| `-f`, `--follow`        | Follow log output. Allow to view the logs dynamically |
| `-s`, `--severity=I`    | Shows logs filtered by severity level.<br> Supported levels are (from low to high): D - Debug, I - Info (default), W - Warning, E - Error, F - Fatal.<br><br> The output includes both the specified severity level and all higher ones |
| `--timezone`=TIMEZONE   | Timezone of the log output. <br>Supported values: `UTC` (default), `local` or the timezone in the [IANA timezone format](https://en.wikipedia.org/wiki/Tz_database) (e.g. `America/New_York`)
| `-i`, `--opid=OPID`     | Show logs for an operation in progress. The operation is identified by the OpID |
| `-x`, `--extra`         | Show extra data in the text format |

Find the usage examples in [Viewing backup logs](../usage/logs.md).

??? admonition "Logs output"
  
    ```json
    [
      {
        "t": "",
        "s": 3,
        "rs": "rs0",
        "node": "example.mongodb.com:27017",
        "e": "",
        "eobj": "",
        "ep": {
          "T": 0,
          "I": 0
        },
        "msg": "listening for the commands"
      },
      ....
    ]
    ```

## pbm oplog-replay

Allows to replay the oplog on top of any backup: logical, physical, storage level snapshot (like EBS-snapshot) and restore it to a specific point in time.

To learn more about the usage, refer to Point-in-Time Recovery oplog replay.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm oplog-replay [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `start=timestamp`       | The start time for the oplog replay. |
| `end=timestamp`         | The end time for the oplog replay.   |
| `--replset-remapping`   | Maps the replica set names for the oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`. |
| `-w`, `--wait`          | Wait for the oplog replay operation to complete. |
| `--wait-time`  | The time to wait for PBM to report the status of the oplog replay execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.| 


## pbm profile add

Saves an external storage configuration to PBM. This information is defined via configuration profiles. 

To learn more about configuration profiles, see [Multiple storages for backups](../features/multi-storage.md).

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm profile add [<flags>] <profile-name> <path/to/profile.yaml>
``` 

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| --sync                  | Add a profile defining an external storage and sync the backup list from this storage|
|--wait                   | Wait for the profile to be added. The flag blocks the shell session.|
| `--wait-time`  | The time to wait for PBM to report the status of adding the profile and backup sync. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.|
|-o, --out=text           | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json`|

??? admonition "Add profile"

    ```json
    {
      "msg": "OK"
    }
    ```

## pbm profile list

Provides information about [configuration profiles](../features/multi-storage.md#configuration-profiles) added to PBM. 

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm profile list [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-o`, `--out=text`      | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |

??? admonition "List profiles"

    ```json
    {
      "profiles": [
        {
          "name": "test1",
          "profile": true,
          "storage": {
            "type": "filesystem",
            "filesystem": {
              "path": "/tmp/local_backups"
            }
          }
        }
      ]
    }
    ```

## pbm profile remove

Removes the specified configuration profile from PBM.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm profile remove <profile-name> [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `--wait`                | Wait for the profile to be removed. The flag blocks the shell session.|
| `--wait-time`  | The time to wait for PBM to report the status of the profile removal. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.|
| `-o`, `--out=text`      | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |

??? admonition "Delete profile"

    ```json
    {
      "msg": "OK"
    }
    ```

## pbm profile show

Shows the external storage configuration according to the specified configuration profile.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm profile show <profile-name> [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-o`, `--out=text`      | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |

??? admonition "Show profile"

    ```json
    {
      "name": "test1",
      "profile": true,
      "storage": {
        "type": "filesystem",
        "filesystem": {
          "path": "/tmp/local_backups"
        }
      }
    }
    ```

## pbm profile sync

Syncs the backup list from the external storage according to the specified configuration profile.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm profile sync <profile-name> [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `--all`                 | Syncs backup lists from all the storages.|
| `--clear`               | Clears the backup list from the storage. To clear the backup list from a specific storage, pass the profile name. When used with `--all`, clears backup lists from all storages. |
| `--wait`                | Wait for the profile to be synced. The flag blocks the shell session.|
| `--wait-time`  | The time to wait for PBM to report the status of the profile sync. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.|
| `-o`, `--out=text`      | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |


## pbm restore

Restores database from a specified backup / to a specified point in time. Depending on the backup type, makes either logical, physical, or a snapshot-based restore.

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm restore [<flags>] [<backup_name>]
```

For more information about using `pbm restore`, see [Restoring a backup](../usage/restore.md).

The command accepts the following flags:

| Flag                | Description                           |
| ------------------- | ------------------------------------- |
| `--external`        | Indicates the backup as the one made outside PBM (for example, snapshot-based)       |
| `--time=TIME`       | Restores the database to the specified point in time. Available for logical restores and if [Point-in-time recovery](../features/point-in-time-recovery.md) is enabled. |
| `-w`                | Wait for the restore to finish. The flag blocks the shell session. |
| `--wait-time`  | The time to wait for PBM to report the status of the restore execution. Use this flag together with the `--wait` flag. You can specify the duration in minutes or hours (e.g. 5m, 1h). <br><br>When not set, PBM waits till the command executes. <br><br>If it takes longer than the defined waiting time to execute the command, PBM prints the `Operation is in progress. Check pbm status and logs` error message and unblocks the shell session. The `pbm-agent` continues to execute the command enabling you to track its progress via the `pbm status` command. Available starting with version 2.6.0.|
| `-o`, `--out=text`  | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |
| `--base-snapshot`   | Restores the database from a specified backup to the specified point in time. Without this flag, the most recent backup preceding the timestamp is used for point in recovery. Available in Percona Backup for MongoDB starting from version 1.6.0.<br><br> In version 2.3.0, this flag is optional for [point-in-time recovery from physical backups](../usage/pitr-physical.md). <br><br> In version 2.2.0, this flag is mandatory for making a [point-in-time recovery from physical backups](../usage/pitr-physical.md). Without it, PBM looks for a logical backup to restore from.|
| `--replset-remapping`| Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|
| `--ns=<database.collection>`| Restores the specified namespace(s) - databases and collections. To restore all collections in the database, specify the values as `--ns=<database.*>`. The `--ns` flag accepts several namespaces as the comma-separated list. For example, `--ns=db1.*,db2.coll2,db3.coll1,db3.collX`|
| `--with-users-and-roles` | Restores users and roles created in custom databases during selective restore. Use this flag with the `--ns` flag. Available starting with version 2.5.0.| 
| `-c`, `--config`     | The path to the `mongod.conf` file |
| `--num-parallel-collections`| Sets the number of collections to process in parallel during a specific logical restore. When undefined, `pbm-agent` processes the number of parallel collections defined for the `restore.numParallelCollections` configuration parameter. If that is undefined,  the default number of collections is the half of the number of logical CPUs. Available starting with version 2.7.0.|
| `--num-insertion-workers-per-collection`| Specifies the number of insertion workers to run concurrently per collection. Increasing the number for large import may increase the speed of the import. Available starting with version 2.8.0.|
| `--ns-from`="database.collection" |Specifies the name of the collection you want to restore under a new name. It indicates the collection present in the backup that you are targeting for restoration. Available starting with version 2.8.0.|
| `--ns-to`="database.collection" | Specifies the new name for the collection you are restoring from the original one. Available starting with version 2.8.0.|
| `--fallback-enabled` | Enables the use of fallback directory for a physical restore. At the restore start, PBM copies the contents of the `dbPath` to a special `.fallback` directory. If a restore is successful, PBM deletes the `.fallback` directory. If the restore ends with an error, PBM triggers the fallback procedure, deleting the downloaded backup files from `dbPath` and copying files from the  `.fallback` directory back, thus reverting the cluster to its initial state before the restore. Disabled by default. Available starting with version 2.10.0|
| `--allow-partly-done` | Instructs how PBM handles restores with the "partlyDone" status. When enabled, PBM considers restore successful and the failed nodes receive the data via the initial sync. At least one node on each shard must restore successfully to serve as the data source for the remaining nodes. When all nodes within a replica set fail to restore, the restore is considered as failed. <br>When disabled, PBM starts the fallback operation if you use  `--allow-partly-done` together with `--fallback-enabled`. Otherwise, the restore won't start. <br><br>Enabled by default. Available starting with version 2.10.0. <br><br> Note that when you restore from backups made with PBM versions before 2.10.0, you must not set the `--allow-partly-done` flag to `false`: PBM automatically disables the `--fallback-enabled` setting, if it is enabled, and cannot start the restore at all because these two flags cannot be disabled simultaneously |


??? "Restore output"

    ```json
    {
       "name": "<restore_name>"
       "snapshot": "<backup_name>"
    }
    ```

??? "Point-in-time restore"

    ```json
    {
      "name":"<restore_name>",
      "point-in-time":"<backup_name>"
    }
    ```
## pbm restore-finish

Instructs PBM to complete the snapshot-based physical restore. Must be run after running `pbm restore --external`. To learn more, refer to [API for snapshot-based physical backups](../features/snapshots.md).

The command has the following syntax:

```{.bash data-prompt="$"}
$ pbm restore-finish <restore_name> [flags]
```

The command accepts the following flags:

| Flag                | Description                           |
| ------------------- | ------------------------------------- |
| `-c`                | The path to the PBM configuration file. Required to complete the restore.|


## pbm status

Shows the status of Percona Backup for MongoDB. The output provides the following information:

* `pbm-agent` processes version, state and node type it is running on (primary or secondary)
* Currently running backups or restores
* Backups stored in the remote storage and their status
* Point-in-Time Recovery status
* Valid time ranges for point-in-time recovery and the data size

The command accepts the following flags:

| Flag                   | Description                             |
| ---------------------- | --------------------------------------- |
| `-o`, `--out=text`     | Shows the status as either plain text or a JSON object. Supported values: `text`, `json` |
| `-p`, `--priority`     | Shows the node priorities for the backup and point-in-time recovery oplog slicing. Available starting with version 2.6.0. |
| `--replset-remapping`  | Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|
| `-s`, `--sections=SECTIONS` | Shows the status for the specified section. You can pass several flags to view the status for multiple sections. Supported values: cluster, pitr, running, backups. |

??? admonition "Status information"

    ```json
    {
      "backups": {
        "type": "FS",
        "path": "<my-backup-dir>",
        "snapshot": [
           ...
          {
            "name": "<backup_name>",
            "size": 3143396168,
            "status": "done",
            "printStatus": "success",
            "restoreTo": Timestamp,
            "pbmVersion": "2.10.0",
            "type": "logical",
            "src": ""
      },
          },
        ],
        "pitrChunks": {
          "pitrChunks": [
             ...
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp
              }
            },
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp (no base snapshot) !!! no backup found
              }
            },
          ],
          "size": 677901884
        }
      },
      "cluster": [
        {
          "rs": "<replSet_name>",
          "nodes": [
            {
              "host": "<replSet_name>/example.mongodb:27017",
              "agent": "<version>",
              "role": "",
              "prio_pitr": "1.0",
              "prio_backup": "1.0",
              "ok": true
            }
          ]
        }
      ],
      "pitr": {
        "conf": true,
        "run": false,
        "error": "Timestamp.000+0000 E [<replSet_name>/example.mongodb:27017] [pitr] <error_message>"
      },
      "running": {
          "type": "backup",
          "name": "<backup_name>",
          "startTS": Timestamp,
          "status": "oplog backup",
          "opID": "6113b631ea9ba5b815fee7c6"
        }
    }
    ```



## pbm version

Shows the version of Percona Backup for MongoDB.

The command accepts the following flags:

| Flag                   | Description                    |
| ---------------------- | ------------------------------ |
| `--short`              | Shows only version info        |
| `--commit`             | Shows only git commit info     |
| `-o`, `--out=text`     | Shows the output as either plain text or a JSON object. Supported values: `text`, `json`|

??? "Version information"

    ```json
    {
      "Version": "1.6.0",
      "Platform": "linux/amd64",
      "GitCommit": "f9b9948bb8201ba1a6400f6558496934a0685efd",
      "GitBranch": "main",
      "BuildTime": "2021-07-28_15:24_UTC",
      "GoVersion": "go1.16.6"
    }
    ```


# `pbm` commands

`pbm` CLI is the command line utility to control the backup system. This page describes `pbm` commands available in Percona Backup for MongoDB.

For how to get started with Percona Backup for MongoDB, see [Initial setup](../initial-setup.md).

## pbm help

Returns the help information about `pbm` commands.

## pbm config

Sets, changes or lists Percona Backup for MongoDB configuration.

The command has the following syntax:

```sh
pbm config [<flags>] [<key>]
```

The command accepts the following flags:

| Flag               | Description                           |
| ------------------ | ------------------------------------- | 
| `--force-resync`   | Resync backup list with the current storage|            
| `--list`           | List current settings                  |
| `--file=FILE`      | Upload the config information from a YAML file   |
| `--set=SET`        | Set a new config option value. Specify the option in the `<key.name=value>` format.                                    |
| `-o`, `--out=text` | Shows the output format as either plain text or a JSON object. Supported values: text, json                      |

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

## pbm backup

Creates a backup snapshot and saves it in the remote backup storage.

The command has the following syntax:

```sh
pbm backup [<flags>]
```

For more information about using `pbm backup`, see [Starting a backup](../usage/start-backup.md)

The command accepts the following flags:

| Flag           | Description                                           |
| -------------- | ----------------------------------------------------- |
| `-t`, `--type` | The type of backup. Supported values: physical, logical (default). When not specified, Percona Backup for MongoDB makes a logical backup. <br> **NOTE**: Physical backups is the technical preview feature [^1].|
| `--compression`| Create a backup with compression. <br> Supported compression methods: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`. Default: `s2` <br> The `none` value means no compression is done during backup. |
| `--compression-level` | Configure the compression level from 0 to 10. The default value depends on the compression method used.  |
| `-o`, `--out=text`    | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |
| `--wait`       | Wait for the backup to finish. The flag blocks the shell session.|

??? "JSON output"

    ```json
    {
      "name": "<backup_name>",
      "storage": "<my-backup-dir>"
    }
    ```

## pbm restore

Restores database from a specified backup / to a specified point in time. Depending on the backup type, makes either logical or physical restore.

The command has the following syntax:

```sh
pbm restore [<flags>] [<backup_name>]
```

For more information about using `pbm restore`, see [Restoring a backup](../usage/restore.md).

The command accepts the following flags:

| Flag                | Description                           |
| ------------------- | ------------------------------------- |
| `--time=TIME`       | Restores the database to the specified point in time. Available for logical restores and if [Point-in-Time Recovery](../usage/point-in-time-recovery.md) is enabled. |
| `-w`                | Wait for the restore to finish. The flag blocks the shell session. |
| `-o`, `--out=text`  | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json` |
| `--base-snapshot`   | Restores the database from a specified backup to the specified point in time. Without this flag, the most recent backup preceding the timestamp is used for point in recovery. Available in Percona Backup for MongoDB starting from version 1.6.0.|
| `--replset-remapping`| Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|

??? "Restore output"

    ```json
    {
      "snapshot": "<backup_name>"
    }
    ```

??? "Point-in-time restore"

    ```json
    {
      "point-in-time": "<backup_name>"
    }
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

## pbm list

Provides the list of backups. In versions 1.3.4 and earlier, the command lists all backups and their states. Backup states are the following:

* In progress - A backup is running
* Canceled - A backup was canceled
* Error - A backup was finished with an error
* No status means a backup is complete

As of version 1.4.0, only successfully completed backups are listed. To view currently running backup information, run [`pbm status`](#pbm-status).

When Point-in-Time Recovery is enabled, the `pbm list` also provides the list of valid time ranges for recovery and point-in-time recovery status.

The command has the following syntax:

```sh
pbm list [<flags>]
```

The command accepts the following flags:

| Flag                | Description                      |
| ------------------- | -------------------------------- |
| `--restore`         | Shows last N restores.           |
| `--size=0`          | Shows last N backups.            |
| `-o`, `--out=text`  | Shows the output format as either plain text or a JSON object. Supported values: `text`, `json`                 |
| `--unbacked`        | Shows Point-in-Time Recovery oplog slices that were saved without the base backup snapshot. Available starting with version 1.8.0.|
| `--replset-remapping` | Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|

??? "List of backups"

    ```json
    {
      "snapshots": [
        {
          "name": "<backup_name>",
          "status": "done",
          "completeTS": Timestamp,
          "pbmVersion": "1.6.0"
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
 
    ```json
    [
      {
        "start": Timestamp,
        "status": "done",
        "type": "snapshot",
        "snapshot": "<backup_name>",
        "name": "2021-07-26T10:08:54.0867213Z"
      },
      {
        "start": Timestamp,
        "status": "done",
        "type": "pitr",
        "snapshot": "<backup_name>",
        "point-in-time": Timestamp,
        "name": "2021-07-26T11:09:53.7500545Z"
      }
    ]
    ```

## pbm delete-backup

Deletes the specified backup snapshot or all backup snapshots that are older than the specified time. The command deletes backups that are not running regardless of the remote backup storage being used.

The following is the command syntax:

```sh
pbm delete-backup [<flags>] [<name>]
```

The command accepts the following flags:

| Flag                     | Description             |
| ------------------------ | ----------------------- |
| `--older-than=TIMESTAMP` | Deletes backups older than date / time specified in the format:<br> - `%Y-%M-%DT%H:%M:%S` (e.g. 2020-04-20T13:13:20) or <br> - `%Y-%M-%D` (e.g. 2020-04-20)|
| `--force`                | Forcibly deletes backups without asking for user’s confirmation  |

## pbm delete-pitr

Deletes oplog slices produced for Point-in-Time Recovery.

The command has the following syntax:

```sh
pbm delete-pitr [<flags>]
```

The command accepts the following flags:

| Flag                     | Description               |
| ------------------------ | ------------------------- |
| `-a`, `--all`            | Deletes all oplog         |
| `--older-than=TIMESTAMP` | Deletes oplog slices older than date / time specified in the format: <br> - `%Y-%M-%DT%H:%M:%S` (e.g. 2020-04-20T13:13:20) or <br> - `%Y-%M-%D` (e.g. 2020-04-20) <br><br> When you specify a timestamp, Percona Backup for MongoDB rounds it down to align with the completion time of the closest backup snapshot and deletes oplog slices that precede this time. Thus, extra slices remain. This is done to ensure oplog continuity. To illustrate, the PITR time range is `2021-08-11T11:16:21 - 2021-08-12T08:55:25` and backup snapshots are: <br><br> `2021-08-12T08:49:46Z 13.49MB [complete: 2021-08-12T08:50:06]` <br> `2021-08-11T11:36:17Z 7.37MB [complete: 2021-08-11T11:36:38]`<br> <br> Say you specify the timestamp `2021-08-11T19:16:21`. The closest backup is `2021-08-11T11:36:17Z 7.37KB [complete: 2021-08-11T11:36:38]`. PBM rounds down the timestamp to `2021-08-11T11:36:38` and deletes all slices that precede this time. As a result, your PITR time range is `2021-08-11T11:36:38 - 2021-08-12T09:00:25`. <br><br> **NOTE**: Percona Backup for MongoDB doesn’t delete the oplog slices that follow the most recent backup. This is done to ensure point in time recovery from that backup snapshot. For example, if the snapshot is `2021-07-20T07:05:23Z [complete: 2021-07-21T07:05:44]` and you specify the timestamp `2021-07-20T07:05:45`, Percona Backup for MongoDB deletes only slices that were made before `2021-07-20T07:05:23Z`.|
| `--force`                | Forcibly deletes oplog slices without asking a user’s confirmation  |
| `-o`, `--out=json`       | Shows the output as either the plain text (default) or a JSON object. Supported values: `text`, `json`.   |


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


## pbm status

Shows the status of Percona Backup for MongoDB. The output provides the following information:

* `pbm-agent` processes version and state,
* currently running backups or restores
* backups stored in the remote storage
* Point-in-Time Recovery status
* Valid time ranges for point-in-time recovery and the data size

The command accepts the following flags:

| Flag                   | Description                             |
| ---------------------- | --------------------------------------- |
| `-o`, `--out=text`     | Shows the status as either plain text or a JSON object. Supported values: `text`, `json` |
| `-s`, `--sections=SECTIONS` | Shows the status for the specified section. You can pass several flags to view the status for multiple sections. Supported values: cluster, pitr, running, backups. |
| `--replset-remapping`  | Maps the replica set names for the data restore / oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`|

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
            "completeTS": Timestamp,
            "pbmVersion": "1.6.0"
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
              "agent": "v1.6.0",
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

## pbm logs

Shows log information from all `pbm-agent` processes.

The command has the following syntax:

```sh
pbm logs [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-t`, `--tail=20`       | Shows last N entries. By default, the output shows last 20 entries. <br> `0` means to show all log messages. |
| `-e`, `--event=EVENT`   | Shows logs filtered by a specified event. Supported events:<br> - backup<br> - restore <br> - resyncBcpList <br> - pitr <br> - pitrestore <br> - delete <br>  |
| `-o`, `--out=text`      | Shows log information as text (default) or in JSON format. <br> Supported values: `text`, `json` |
| `-n`, `--node=NODE`     | Shows logs for a specified node or a replica set.<br> Specify the node in the format `replset[/host:port]` |
| `-s`, `--severity=I`    | Shows logs filtered by severity level.<br> Supported levels are (from low to high): D - Debug, I - Info (default), W - Warning, E - Error, F - Fatal.<br><br> The output includes both the specified severity level and all higher ones |
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

```sh
pbm oplog-replay [<flags>]
```

The command accepts the following flags:

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `start=timestamp`       | The start time for the oplog replay. |
| `end=timestamp`         | The end time for the oplog replay.   |
| `--replset-remapping`   | Maps the replica set names for the oplog replay. The value format is `to_name_1=from_name_1,to_name_2=from_name_2`. |


[^1]: Tech Preview Features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.

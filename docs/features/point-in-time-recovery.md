# Point-in-time recovery

!!! admonition "Version added: [1.3.0](../release-notes/1.3.0.md)"

??? admonition "Implementation history"

    The following table lists the changes in the implementation of point-in-time recovery and the versions that introduced those changes:

    |Version | Description |
    |--------|-------------|
    | [1.3.0](../release-notes/1.3.0.md)   | Initial implementation of point-in-time recovery|
    | [1.6.0](../release-notes/1.6.0.md)   | Ability to change duration of an oplog span|
    | [1.7.0](../release-notes/1.7.0.md)   | Added compression to oplog slices|
    | [2.3.0](../release-notes/2.3.0.md)   | Support of any type of base backup|
    | [2.4.0](../release-notes/2.4.0.md)   | Oplog slicing in parallel with backups|
    |[2.6.0](../release-notes/2.6.0.md))   | Adjust node priority for oplog slices|

Point-in-time recovery is restoring a database up to a specific timestamp. This includes restoring the data from a backup snapshot and replaying all events that occurred to this data up to a specified time from [oplog slices](#oplog-slicing). 

| Advantages                     | Disadvantages                   |
| ------------------------------ | ------------------------------- |
| Helps you prevent data loss during a disaster such as a crashed database, accidental data deletion or drop of tables, and unwanted update of multiple fields instead of a single one. | Restore takes longer since it requires you to restore the backup and then replay oplog events on top of it.|

## Enable point-in-time recovery

Set the `pitr.enabled` configuration option to `true`.

=== ":octicons-file-code-24: Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.enabled=true
     ```

=== ":material-console: Configuration file"

     ```yaml
     pitr:
       enabled: true
     ```

The `pbm-agent` starts [saving consecutive slices of the oplog](#oplog-slicing) periodically. The `pbm-agent` to save oplog slices is randomly selected among the nodes according to their priority, whether it is the default or [user-defined one](#adjust-node-priority-for-oplog-slices). 

You can manage the `pbm-agent` election by assigning a priority value to the `mongod` nodes. See the [Adjust node priority for oplog slices](#adjust-node-priority-for-oplog-slices) for details.


[Restore to a point-in-time](../usage/pitr-tutorial.md){ .md-button .md-button }

## Oplog slicing

To start saving [oplog slices](../reference/glossary.md#oplog), the following preconditions must be met:

=== ":material-data-matrix: Logical backups"

    * A full backup snapshot is required. Starting with version [2.3.0](../release-notes/2.3.0.md), it can be a logical, a physical or an incremental backup. Make sure that a [backup exists](../usage/list-backup.md). See the [Make a backup](../usage/start-backup.md) guide to make a backup snapshot.
    * Point-in-time recovery routine is [enabled](#enable-point-in-time-recovery). 

=== ":material-database-refresh-outline: Physical backups"

    Point-in-time recovery routine is [enabled](#enable-point-in-time-recovery). 
    

If you just enabled point-in-time recovery, it requires 10 minutes for the first slice to appear in the [`pbm list`](../reference/pbm-commands.md#pbm-list) output.

!!! important

    **For in MongoDB 5.0 and higher versions**

    If you [reshard :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/) a collection, make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

    **For MongoDB 8.0 and higher versions**

    If you [unshard a collection :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/reference/command/unshardCollection/), make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

Starting with version [2.4.0](../release-notes/2.4.0.md), oplog slicing runs as follows:

* **Logical backups** 

    Before backup starts, the point-in-time recovery routine is automatically disabled. A backup routine creates oplog slices during the backup creation. After the backup is complete, the point-in-time recovery routine is re-enabled automatically. It copies the slices taken during the backup and continues oplog slicing from the latest timestamp. 

* **Physical backups** 

    During a physical backup, the point-in-time recovery routine is not disabled and continues to save oplog slices in parallel with a backup snapshot operation. 


Thus, if a backup snapshot is large and takes hours to make, all oplog events are saved for it, ensuring point-in-time recovery to any timestamp.

### Oplog duration

!!! admonition "Version added: [1.6.0](../release-notes/1.6.0.md)"

By default, a slice covers a 10-minute span of oplog events. It can be shorter if point-in-time recovery is disabled or interrupted by the start of a backup snapshot operation.

You can change the duration of an oplog span via the configuration file. Specify the new value (in minutes) for the `pitr.oplogSpanMin` option.

=== ":octicons-file-code-24: Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.oplogSpanMin=5
     ```

=== ":material-console: Configuration file"

     ```yaml
     pitr:
       oplogSpanMin: 5
     ```

If you set the new duration when the `pbm-agent` is making an oplog slice, the slice’s span is updated right away.

If the new duration is shorter, this triggers the `pbm-agent` to make a new slice with the updated span immediately. If the new duration is larger,  the `pbm-agent` makes the next slice with the updated span in its scheduled time.

### Adjust node priority for oplog slices

!!! admonition "Version added: [2.6.0](../release-notes/2.6.0.md)"

Before version 2.6.0, the `pbm-agent` to save oplog slices is selected randomly across replica set members. 

Starting with version 2.6.0, you can control from what node to save oplog slices by assigning the priority to the desired nodes via the configuration file. For example, you can ensure that both backups and oplog slices are taken from the nodes in a specific data center as defined in the organization's regulations. Or, you can reduce network latency by making backups and / or oplog slices from nodes in geographically closest locations.  

Node priority for oplog slices is handled similarly to the [node priority for making backups](../usage/start-backup.md#adjust-node-priority-for-backups), yet it is independent from it. Thus, you can assign a different priority for backups and oplog slices for the same node. Or, adjust only the priority for oplog slices, leaving the default one for backups. 

PBM then handles both processes according to their priority.

The default node priority for oplog slices is the same as for making backups:

* hidden node - priority 2
* secondary nodes - priority 1
* primary node - priority 0.5

To redefine it, specify the new priority for the [`pitr.priority`](../reference/pitr-options.md#pitrnodepriority) option in the configuration file:

```yaml
pitr:
  enabled: true
  priority:
    "rs1:27017": 1
    "rs2:27018": 2
    "rs3:27019": 1
```

The format of the priority array is `<hostname:port>:<priority>`.

!!! important

    As soon as you adjust node priorities in the configuration file, it is assumed that you take manual control over them. The default rule to prefer secondary nodes over primary stops working.

To define priority in a sharded cluster, you can either list all nodes or specify priority for one node in each shard and config server replica set. The `hostname` and `port` uniquely identifies a node so that Percona Backup for MongoDB recognizes where it belongs to and grants the priority accordingly.

Note that if you listed only specific nodes, the remaining nodes will be automatically assigned priority `1.0`. For example, you assigned priority `2.5` to only one secondary node in every shard and config server replica set of the sharded cluster.

```yaml
pitr:
  enabled: true
  priority:
    "localhost:27027": 2.5  # config server replica set
    "localhost:27018": 2.5  # shard 1
    "localhost:28018": 2.5  # shard 2
```

The remaining secondaries and the primary nodes in the cluster receive priority `1.0`.

To check the priorities, run the `pbm status` command with the  `--priority` flag.

```{.bash data-prompt="$"}
$ pbm status --priority
```

??? example "Sample output"

    ```{.text .no-copy}
    Cluster:
    ========
    rs1:
      - rs1/rs101:27017 [S], Bkp Prio: [1.0], PITR Prio: [2.5]: pbm-agent [v{{release}}] OK
      - rs1/rs102:27017 [P], Bkp Prio: [0.5], PITR Prio: [2.0]: pbm-agent [v{{release}}] OK
      - rs1/rs103:27017 [S], Bkp Prio: [1.0], PITR Prio: [1.0]: pbm-agent [v{{release}}] OK
    ```

PBM saves oplog slices from the node with the highest priority. If this node is not responding, it selects the next priority node. If there are several nodes with the same priority, one of them is randomly elected for saving oplog slices.


### Compressed oplog slices 

!!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)"

The oplog slices are saved with the `s2` compression method by default. You can specify a different compression method via the configuration file. Specify the new value for the [`pitr.compression`](../reference/pitr-options.md#pitrcompression) option.

=== ":octicons-file-code-24: Command line"

     ```{.bash data-prompt="$"}
     $ pbm config --set pitr.compression=gzip
     ```

=== ":material-console: Configuration file"

     ```yaml
     pitr:
       compression: gzip
     ```

Supported compression methods are: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`.

Additionally, you can override the compression level used by the compression method by setting the [`pitr.compressionLevel`](../reference/pitr-options.md#pitrcompressionlevel) option. The default values differ for each compression level. 

Note that the higher value you specify, the more time and computing resources it will take to compress the data.

!!! note 

    You can use different compression methods for backup snapshots and point-in-time recovery slices. However, backup snapshot-related oplog is compressed with the same compression method as the backup itself.

### View oplog slices

The oplog slices are stored in the `pbmPitr` subdirectory in the [remote storage defined in the config](../details/storage-configuration.md#supported-storage-types). A slice name reflects the start and end time this slice covers.

The [`pbm list`](../reference/pbm-commands.md#pbm-list) output includes the following information:

* Backup snapshots. As of version 1.4.0, it also shows the completion time (renamed to the `restore_to_time` in version 2.0.0)
* Valid time ranges for recovery
* Point-in-time recovery status

   ```{.bash data-prompt="$"}
   $ pbm list

     2021-08-04T13:00:58Z [restore_to_time: 2021-08-04T13:01:23Z]
     2021-08-05T13:00:47Z [restore_to_time: 2021-08-05T13:01:11Z]
     2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09Z]
     2021-08-06T08:03:43Z [restore_to_time: 2021-08-06T08:04:08Z]
     2021-08-06T08:18:17Z [restore_to_time: 2021-08-06T08:18:41Z]

   PITR <off>:
     2021-08-04T13:01:24 - 2021-08-05T13:00:11
     2021-08-06T08:03:10 - 2021-08-06T08:18:29
     2021-08-06T08:18:42 - 2021-08-06T08:33:09
   ```



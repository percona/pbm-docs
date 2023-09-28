# Make a backup

## Before you start

1. [Install](../installation.md) and [set up Percona Backup for MongoDB](../install/initial-setup.md)
2. Check that `pbm agent` is running with the [`pbm status`](../reference/pbm-commands.md#pbm-status) command

## Make a backup 

=== "Logical"

     To make a backup, run the following command:

     ```{.bash data-prompt="$"}
     $ pbm backup --type=logical
     ```
     
     Logical backup is the default one so you can bypass the `--type` flag. 

     During *logical* backups, Percona Backup for MongoDB copies the actual data to the backup storage.

     Starting with version 2.0.0, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

     Multi-format is now the default data format since it allows [selective restore](restore.md). Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB.

=== "Physical"
     
    !!! admonition "Version added: [1.7.0](../release-notes/1.7.0.md)" 

     ```{.bash data-prompt="$"}
     $ pbm backup --type=physical
     ```

     During a *physical* backup, Percona Backup for MongoDB stops [point-in-time recovery oplog slicing](../features/point-in-time-recovery.md#oplog-slicing) if it's enabled, copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage.

=== "Selective"

    !!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

    Before you start, read about [selective backups known limitations](../features/selective-backup.md#known-limitations-of-selective-backups-and-restores).

    To make a selective backup,  run the `pbm backup` command and provide the value for the `--ns` flag in the format `<database.collection>`. The `--ns` flag value is case sensitive. For example, to back up the "Payments" collection, run the following command:

     ```{.bash data-prompt="$"}
     $ pbm backup --ns=staff.Payments
     ```

     To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

     ```{.bash data-prompt="$"}
     $ pbm backup --ns=Invoices.*
     ```

     During the backup process, Percona Backup for MongoDB stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

     Multi-format is now the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of Percona Backup for MongoDB. 

=== "Incremental"
    
    !!! admonition "Version added: [2.0.3](../release-notes/2.0.3.md)"

    Before you start, read more about [incremental backup](../features/incremental-backup.md#considerations).

    To start incremental backups, first make a full incremental backup. It will serve as the base for subsequent incremental backups:

    ```{.bash data-prompt="$"} 
    $ pbm backup --type incremental --base
    ```

    The `pbm-agent` starts tracking the incremental backup history to be able to calculate and save the difference in data blocks. After that you can run regular incremental backups:

    ```{.bash data-prompt="$"}
    $ pbm backup --type incremental
    ```

    The incremental backup history looks like this:

    ```{.bash .no-copy} 
    Snapshots:
        2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
        2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
        2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
        2022-11-24T14:45:53Z 220.13GB <incremental, base> [restore_to_time: 2022-11-24T14:45:55Z]
    ```

=== "Snapshot-based"

    See [snapshot-based backups](../features/snapshots.md#make-a-backup).



### Compressed backups

By default, Percona Backup for MongoDB uses the `s2` compression method when making a backup.

You can start a backup with a different compression method by passing the `--compression` flag to the **pbm backup** command.

For example, to start a backup with `gzip` compression, use the following command:

```{.bash data-prompt="$"}
$ pbm backup --compression=gzip
```

Supported compression types are: `gzip`, `snappy`, `lz4`, `pgzip`, `zstd`.  The `none` value means no compression is done during backup.

As of version 1.7.0, you can configure the compression level for backups. Specify the value for the [`--compression-level`](../reference/backup-options.md#backupcompressionlevel) flag. 

Default compression levels differ per compression method used. 

The following table shows available compression levels per compression method:

| Compression method   | Supported compression levels | Default
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - fastest speed, 2 - default, 3 - better compression, 4 - best compression | 2
| `snappy`             | no levels|
| `lz4`                | From 1 (fastest) to 16 | 1
| `gzip` and `pgzip`   | -1 - default compression, 0 - no compression, 1 - best speed, 9 - best compression| -1

Note that the higher value you specify, the more time and computing resources it will take to compress the data.

## Backups in sharded clusters

!!! important "For PBM v1.0 (only)"

    Before running pbm backup on a cluster, stop the balancer.

In sharded clusters, one of the **pbm-agent** processes for every shard and the config server replica set writes backup snapshots  into the remote backup storage directly. For logical backups, `pbm-agents` also write oplog slices. To learn more about oplog slicing, see Point-in-Time Recovery.

The `mongos` nodes are not involved in the backup process.

The following diagram illustrates the backup flow.

![image](../_images/pbm-backup-shard.png)

!!! important

    If you reshard a collection in MongoDB 5.0 and higher versions, make a fresh backup to prevent data inconsistency and restore failure.

### Adjust node priority for backups

In Percona Backup for MongoDB prior to version 1.5.0, the `pbm-agent` to do a backup is elected randomly among secondary nodes in a replica set. In sharded cluster deployments, the `pbm-agent` is elected among the secondary nodes in every shard and the config server replica sets. If no secondary node responds in a defined period, then the `pbm-agent` on the primary node is elected to do a backup.

As of version 1.5.0, you can influence the `pbm-agent` election by assigning a priority to `mongod` nodes in the Percona Backup for MongoDB [configuration file](../reference/config.md).

```yaml
backup:
  priority:
    "localhost:28019": 2.5
    "localhost:27018": 2.5
    "localhost:27020": 2.0
    "localhost:27017": 0.1
```

The format of the priority array is `<hostname:port>`:`<priority>`.

To define priority in a sharded cluster, you can either list all nodes or specify priority for one node in each shard and config server replica set. The `hostname` and `port` uniquely identifies a node so that Percona Backup for MongoDB recognizes where it belongs to and grants the priority accordingly.

Note that if you listed only specific nodes, the remaining nodes will be automatically assigned priority `1.0`. For example, you assigned priority `2.5` to only one secondary node in every shard and config server replica set of the sharded cluster.

```yaml
backup:
  priority:
    "localhost:27027": 2.5  # config server replica set
    "localhost:27018": 2.5  # shard 1
    "localhost:28018": 2.5  # shard 2
```

The remaining secondaries and the primary nodes in the cluster receive priority `1.0`.

The `mongod` node with the highest priority makes the backup. If this node is unavailable, the next priority node is selected. If there are several nodes with the same priority, one of them is randomly elected to make the backup.

If you haven’t listed any nodes for the `priority` option in the config, the nodes have the default priority for making backups as follows:

* hidden nodes - priority 2.0
* secondary nodes - priority 1.0
* primary node - priority 0.5

!!! important

    As soon as you adjust node priorities in the configuration file, it is assumed that you take manual control over them. The default rule to prefer secondary nodes over primary stops working.

This ability to adjust node priority helps you manage your backup strategy by selecting specific nodes or nodes from preferred data centers. In geographically distributed infrastructures, you can reduce network latency by making backups from nodes in geographically closest locations.

## Next steps

* [List backups](../usage/list-backup.md)
* [Make a restore](restore.md)

## Useful links

* [Backup and restore types](../features/backup-types.md)
* [Schedule backups](../usage/schedule-backup.md)


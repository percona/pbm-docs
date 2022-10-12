# Start a backup

To start a backup, run the following command:

```sh
pbm backup --type=TYPE
```

!!! admonition "Version added: 1.7.0" 

You can specify what type of a backup you wish to make: *physical* or *logical*.

When *physical* backup is selected, Percona Backup for MongoDB copies the contents of the `dbpath` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage.

During *logical* backups, Percona Backup for MongoDB copies the actual data to the backup storage. When no `--type` flag is passed, Percona Backup for MongoDB makes a logical backup.

For more information about backup types, see [Backup and restore types](../details/backup-types.md).

## Compressed backups

By default, Percona Backup for MongoDB uses the `s2` compression method when making a backup.

You can start a backup with a different compression method by passing the `--compression` flag to the **pbm backup** command.

For example, to start a backup with `gzip` compression, use the following command:

```sh
pbm backup --compression=gzip
```

Supported compression types are: `gzip`, `snappy`, `lz4`, `pgzip`, `zstd`.  The `none` value means no compression is done during backup.

As of version 1.7.0, you can configure the compression level for backups. Specify the value for the `--compression-level` flag. Note that the higher value you specify, the longer it takes to compress / retrieve the data.

## Backups in sharded clusters

!!! important "For PBM v1.0 (only)"

    Before running pbm backup on a cluster, stop the balancer.

In sharded clusters, one of **pbm-agent** processes for every shard and the config server replica set writes backup snapshots  into the remote backup storage directly. For logical backups, `pbm-agents` also write oplog slices. To learn more about oplog slicing, see Point-in-Time Recovery.

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

The `mongod` node with the highest priority makes the backup. If this node is unavailable, next priority node is selected. If there are several nodes with the same priority, one of them is randomly elected to make the backup.

If you haven’t listed any nodes for the `priority` option in the config, the nodes have the default priority for making backups as follows:

* hidden nodes - priority 2.0
* secondary nodes - priority 1.0
* primary node - priority 0.5

!!! important

    As soon as you adjust node priorities in the configuration file, it is assumed that you take manual control over them. The default rule to prefer secondary nodes over primary stops working.

This ability to adjust node priority helps you manage your backup strategy by selecting specific nodes or nodes from preferred data centers. In geographically distributed infrastructures, you can reduce network latency by making backups from nodes in geographically closest locations.


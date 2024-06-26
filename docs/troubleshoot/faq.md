# FAQ

## What’s the difference between PBM and `mongodump`?

`mongodump` is a "logical" backup solution only while Percona Backup for MongoDB supports both logical and physical backups. Both solutions have equal performance for non-sharded replica sets. However, as opposed to `mongodump`, Percona Backup for MongoDB allows you to achieve the following goals:

* Make consistent backups and restores in sharded clusters.
* Backup / restore both the whole data set and specific namespaces —— databases and collections. (See [Selective backup and restore](../features/selective-backup.md) for more information.)
* Restore your database to a specific point in time.
* Run backups / restores on each replica set in parallel while `mongodump` runs in one process on `mongos` node.

## Why does Percona Backup for MongoDB use UTC timezone instead of server local timezone?

`pbm-agents` use UTC time zone by design. The reason behind this is to avoid user misunderstandings when replica set / cluster nodes are distributed geographically in different time zones.

Starting with version 2.0.1, you can change the time zone for ``pbm logs`` output.

## Can I restore a single collection with Percona Backup for MongoDB?

Yes. Starting with version 2.0.0, you can restore a single collection with Percona Backup for MongoDB. This functionality is available for logical backups and restores only. To learn more, see [Selective backup and restore](../features/selective-backup.md).

## Can I back up specific shards in a cluster?

No, since this would result in backups with inconsistent timestamps across the cluster. Such backups would be invalid for restore.

Percona Backup for MongoDB backs up the whole state of a sharded cluster, and this guarantees data consistency during the restore.

## Do I need to stop the balancer for PITR restore?

Yes. The preconditions for both Point-in-Time Recovery restore and regular restore are the same:


1. In a sharded cluster, stop the balancer.


2. Make sure no writes are made to the database during restore. This ensures data consistency.


3. Disable Point-in-Time Recovery if it is enabled. This is because oplog slicing and restore are exclusive operations and cannot be run together. Note that oplog slices made after the restore and before the next backup snapshot become invalid. Make a fresh backup and re-enable Point-in-Time Recovery.

## Can I install PBM on MacBook?

You cannot install PBM on MacBook using the package manager. PBM packages are available and tested for Linux distributions only. However, you can run PBM on MacBook as a Docker container. See the [Run in Docker](../install/docker.md) guide for guidelines. 

## Can I connect PBM to MongoDB with disabled authorization?

While we **don’t recommend** disabling authorization for MongoDB due to security considerations, there might be scenarios (such as testing environments) where you need to connect PBM (Percona Backup Manager) to a MongoDB instance without authentication. Follow these steps to ensure a successful setup:

1. If you’ve set the [`bindIP` :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp) configuration parameter for your `mongod` or `mongos` processes, ensure that the `localhost` value is included in the list of allowed IP addresses. The `pbm-agent` process connects to its local MongoDB node using a standalone type of connection. 
2. Make sure the MongoDB connection URI string for pbm-agents includes `localhost` as part of the host information. 
3. In the MongoDB connection URI string for the PBM CLI, exclude the `authSource` parameter as it otherwise enforces authorization.

# Known limitations for backups and restores

PBM supports various backup and restore types. Some of them have known limitations. This page lists them per backup / restore type and serves as a single source of truth for known limitations.

## Selective backups and restores

1. Only **logical** backups and restores are supported.
2. Selective backups and restores are supported in sharded clusters for non-sharded collections starting with version 2.0.3. Sharded collections are supported starting with version 2.1.0. 
3. Sharded time series collections are not supported.
4. Multi-collection transactions are not yet supported for selective restore. However, if you use them and attempt a selective restore, it may break [ACID](../reference/glossary.md#acid) because not all operations with this transaction are restored. PBM applies oplog events that relate only to the specified namespaces(s). Thus, from the transaction's point of view, the data consistency may be broken.

    For example, you have a transaction that involves collections A and B. When you restore collection A, PBM replays oplog events only for collection A and ignores those related to collection B. As a result, the state of collection B remains unchanged and is no longer consistent with collection A. 
    
5. System collections in ``admin``, ``config``, and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
6. Selective point-in-time recovery is not supported for sharded clusters.
7. Selective backups are not supported for deployments with [config shards :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/core/sharded-cluster-config-servers/#std-label-sharded-cluster-config-server-config-shards) - config server replica sets that also store application data.

## Oplog slicing for point-in-time recovery

Oplog slicing is an integral part of the point-in-time recovery routine that enables you to restore from a backup up to a specific moment. Read more about [point-in-time recovery](point-in-time-recovery.md).

**For in MongoDB 5.0 and higher versions**

If you [reshard :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/) a collection, make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

**For MongoDB 8.0 and higher versions**

If you [unshard a collection :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/reference/command/unshardCollection/), make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

## Oplog replay from arbitrary start time

The oplog replay fails if you rename the entire database or a collection.

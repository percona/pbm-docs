# Known limitations for backups and restores

PBM supports various backup and restore types. Some of them have known limitations. This page lists them per backup / restore type and serves as a single source of truth for known limitations.

## Logical restores of time series collections with expiration

A logical restore, including point-in-time recovery, may fail if the backup contains a time series collection configured with `expireAfterSeconds`. During the restore, MongoDB’s TTL monitor can remove expired time series buckets that PBM still needs for oplog replay. The restore may fail with an error similar to the following:

```text
(Location6781400) Time series bucket document is missing 'control' field
```

Before starting the restore, manually disable the TTL monitor on every data-bearing `mongod` instance in the target deployment. Keep it disabled for the duration of the restore. The [`ttlMonitorEnabled` :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/reference/parameters/#mongodb-parameter-param.ttlMonitorEnabled){:target="_blank"} parameter applies to individual instances, so configure each one separately:
{.power-number}

1. Connect directly to the instance using `mongosh` with `directConnection=true` in the connection string. Do not connect through `mongos`.

2. Check and record the current setting:

    ```sh
    db.adminCommand({ getParameter: 1, ttlMonitorEnabled: 1 })
    ```

3. Disable the TTL monitor:

    ```sh
    db.adminCommand({ setParameter: 1, ttlMonitorEnabled: false })
    ```

4. When the restore ends, restore the recorded setting on each instance, even if the restore fails. For example, if the previous value was `true`, run:

    ```sh
    db.adminCommand({ setParameter: 1, ttlMonitorEnabled: true })
    ```

!!! warning "TTL cleanup"

    Disabling the TTL monitor pauses automatic removal of expired data, including data in system collections. Limit this pause to the restore operation.

    When TTL cleanup resumes, restored data that has already expired becomes eligible for deletion, even if it had not expired at the selected recovery point.

For details about time series expiration, see [Automatic removal for time series collections :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/core/timeseries/timeseries-automatic-removal/){:target="_blank"}.


## Selective backups and restores

1. Only **logical** backups and restores are supported.
2. Selective backups and restores are supported in sharded clusters for non-sharded collections starting with version 2.0.3. Sharded collections are supported starting with version 2.1.0.
3. Sharded time series collections are not supported.
4. Multi-collection transactions are not yet supported for selective restore. However, if you use them and attempt a selective restore, it may break [ACID](../reference/glossary.md#acid) because not all operations with this transaction are restored. PBM applies oplog events that relate only to the specified namespace(s). Thus, from the transaction's point of view, the data consistency may be broken.

    For example, you have a transaction that involves collections A and B. When you restore collection A, PBM replays oplog events only for collection A and ignores those related to collection B. As a result, the state of collection B remains unchanged and is no longer consistent with collection A. 
    
5. System collections in ``admin``, ``config``, and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
6. Selective point-in-time recovery is not supported for sharded clusters.

## Physical backups and restores

Physical restores are not supported for MongoDB instances running without authentication if the PBM agent connects via a non-localhost interface (such as a container hostname or external IP). Because MongoDB restricts the shutdown command to the localhost interface in non-authenticated environments, PBM will be unable to stop the node to perform the restore. To avoid this limitation, ensure that the `PBM_MONGODB_URI` uses a `localhost` connection or enable authentication for your MongoDB deployment.

## Oplog slicing for point-in-time recovery

Oplog slicing is an integral part of the point-in-time recovery routine that enables you to restore from a backup up to a specific moment. Read more about [point-in-time recovery](point-in-time-recovery.md).

**MongoDB 8.0 and higher versions**

If you [unshard a collection :octicons-link-external-16:](https://www.mongodb.com/docs/v8.0/reference/command/unshardCollection/), make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

**MongoDB 5.0 and higher versions**

If you [reshard :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/) a collection, make a fresh backup and re-enable point-in-time recovery oplog slicing to prevent data inconsistency and restore failure.

## Oplog replay from arbitrary start time

The oplog replay fails if you rename the entire database or a collection.

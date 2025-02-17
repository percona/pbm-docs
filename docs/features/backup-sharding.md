# Backups in sharded clusters

!!! important "For PBM v1.0 (only)"

    Before running pbm backup on a cluster, stop the balancer.

In sharded clusters, one of the **pbm-agent** processes for every shard and the config server replica set writes backup snapshots  into the remote backup storage directly. For logical backups, `pbm-agents` also write oplog slices. To learn more about oplog slicing, see Point-in-Time Recovery.

The `mongos` nodes are not involved in the backup process.

The following diagram illustrates the backup flow.

![image](../_images/pbm-backup-shard.png)

!!! important

    If you reshard a collection in MongoDB 5.0 and higher versions or unshard a collection in MongoDB 8.0 and higher versions, make a fresh backup to prevent data inconsistency and restore failure.
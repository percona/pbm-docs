## Preconditions

Run [`pbm status`](../reference/pbm-commands.md#pbm-status) or [`pbm list`](../reference/pbm-commands.md#pbm-list) commands to check that the full backup snapshot exists and there are oplog slices.

## Before you start

Before restoring, prepare the cluster to prevent sharding metadata changes and ensure a consistent restore.
{.power-number}

1. Stop the balancer and `mongos` nodes.
2. Make sure no writes are made to the database during restore. 

## Preconditions

Run [`pbm status`](../reference/pbm-commands.md#pbm-status) or [`pbm list`](../reference/pbm-commands.md#pbm-list) commands to check that the full backup snapshot exists and there are oplog slices.

## Before you start

1. Disable point-in-time recovery. A restore and point-in-time recovery oplog slicing are incompatible operations and cannot be run simultaneously. 

    ```{.bash data-prompt="$"}
    $ pbm config --set pitr.enabled=false
    ```

2. Stop the balancer and `mongos` nodes.
3. Make sure no writes are made to the database during restore. 

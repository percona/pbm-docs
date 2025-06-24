---
glightbox-manual: true
---

# Partially done physical restores

When you restore a physical backup, the operation can result in the following statuses:

- **Done**: The restore completed successfully on all nodes.
- **Error**: The restore failed and could not be completed.
- **Partly-Done**: The restore was successful on at least one node in each replica set, but failed on some nodes. 

## Restore status decision flow

```mermaid
flowchart TD
    A[Check status] --> B{Restore successful on all nodes?}
    B -- Yes --> C["Status <b>Done</b>. Do post-restore steps"]
    B -- No --> D{Is min 1 node in each replica set restored?}
    D -- Yes --> E{Is fallback enabled?}
    D -- No --> F["Status <b>Error</b>. Is fallback enabled?"]
    F -- No --> G[Manual intervention required]
    F -- Yes --> H[Revert to pre-restore state]
    E -- Yes --> I{Is allowPartlyDone enabled?}
    E -- No --> J{Is allowPartlyDone enabled?}
    I -- Yes --> K["Status <b>partlyDone</b>. Do post-restore steps"]
    J -- Yes --> K
    J -- No --> H
```

The decision flow is explained below:

1. **Check the restore status** with `pbm describe-restore <restore_name>` or `pbm status`.
2. **Did the restore succeed on all nodes?**
    - **Yes**: The status is **Done**. Continue with [post-restore steps](#post-restore-steps).
    - **No**: Go to the next step.
3. **Did at least one node in each replica set restore successfully?**
    - **Yes**: Is `fallbackEnabled` set to `true`?
        - **Yes**: Is `allowPartlyDone` enabled?
            - **Yes**: The status is **PartlyDone**. See [Post-restore steps for `partlyDone` restores](#post-restore-steps-for-partlydone-restores).
            - **No**: PBM triggers a fallback procedure and restores your cluster to its pre-restore state. If `fallbackEnabled` is disabled, the restore status is **Error**.
    - **No**: The status is **Error**. You'll need to intervene manually.

## Post-restore steps for `partlyDone` restores

If your restore finishes with the **partlyDone** status, you can still start the cluster and wait for the failed node to receive the data via the initial sync. Here's what you need to do:

1. Check the restore status with `pbm status` or `pbm describe-restore <restore_name>`.
2. Start all `mongod` nodes. The failed nodes will perform initial sync from the healthy nodes.
3. Wait for the nodes to complete initial sync and report the `ready` status.
4. Start `pbm-agents` on every node.
5. Start the balancer and all `mongos` nodes.
6. Make a fresh backup to serve as the new base for future restores.
7. [Enable point-in-time recovery](../features/point-in-time-recovery.md/#enable-point-in-time-recovery) if you need it.
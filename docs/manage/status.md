# Percona Backup for MongoDB status

!!! admonition "Version added: [1.4.0](../release-notes/1.4.0.md)"

You can check the status of Percona Backup for MongoDB running in your MongoDB environment using the [`pbm status`](../reference/pbm-commands.md#pbm-status) command.

```{.bash data-prompt="$"}
$ pbm status
```

The output provides the information about:

* Your MongoDB deployment and `pbm-agents` running in it: to what `mongod` node each agent is connected, the Percona Backup for MongoDB version it runs and the agent’s state

* The currently running backups / restores, if any

* Backups stored in the remote backup storage: backup name, completion time, size and status (complete, canceled, failed)

* [Point-in-time recovery](../features/point-in-time-recovery.md) status (enabled or disabled)

* Valid time ranges for point-in-time recovery and the data size

This simplifies troubleshooting since the whole information is provided in one place.

**Sample output**

```{.bash .no-copy}
pbm status

Cluster:
========
config:
  - config/localhost:27027: pbm-agent v1.3.2 OK
  - config/localhost:27028: pbm-agent v1.3.2 OK
  - config/localhost:27029: pbm-agent v1.3.2 OK
rs1:
  - rs1/localhost:27018: pbm-agent v1.3.2 OK
  - rs1/localhost:27019: pbm-agent v1.3.2 OK
  - rs1/localhost:27020: pbm-agent v1.3.2 OK
rs2:
  - rs2/localhost:28018: pbm-agent v1.3.2 OK
  - rs2/localhost:28019: pbm-agent v1.3.2 OK
  - rs2/localhost:28020: pbm-agent v1.3.2 OK

PITR incremental backup:
========================
Status [OFF]

Currently running:
==================
(none)

Backups:
========
S3 us-east-1 https://storage.googleapis.com/backup-test
   Snapshots:
     2020-12-16T10:36:52Z 491.98KB [restore_to_time: 2020-12-16T10:37:13Z]
     2020-12-15T12:59:47Z 284.06KB [restore_to_time: 2020-12-15T13:00:08Z]
     2020-12-15T11:40:46Z 0.00B [canceled: 2020-12-15T11:41:07Z]
     2020-12-11T16:23:55Z 284.82KB [restore_to_time: 2020-12-11T16:24:16Z]
     2020-12-11T16:22:35Z 284.04KB [restore_to_time: 2020-12-11T16:22:56Z]
     2020-12-11T16:21:15Z 283.36KB [restore_to_time: 2020-12-11T16:21:36Z]
     2020-12-11T16:19:54Z 281.73KB [restore_to_time: 2020-12-11T16:20:15Z]
     2020-12-11T16:19:00Z 281.73KB [restore_to_time: 2020-12-11T16:19:21Z]
     2020-12-11T15:30:38Z 287.07KB [restore_to_time: 2020-12-11T15:30:59Z]
PITR chunks:
     2020-12-16T10:37:13 - 2020-12-16T10:43:26 44.17KB
```
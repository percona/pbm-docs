# pbm-agent

A `pbm-agent` is a process that runs backup, restore, delete, and other operations available with Percona Backup for MongoDB.

A `pbm-agent` instance must run on each `mongod` instance. This includes replica set nodes that are currently secondaries and config server replica set nodes in a sharded cluster.

An operation is triggered when the [`pbm` CLI](../reference/glossary.md#pbm-cli) makes an update to the [PBM Control collection](../reference/glossary.md#pbm-control-collections). All `pbm-agents` monitor changes to the PBM control collections, but only one `pbm-agent` in each replica set will be elected to execute an operation. The elections are done by a random choice among secondary nodes. If no secondary nodes respond, then the `pbm-agent` on the primary node is elected for an operation.

The elected `pbm-agent` acquires a lock for an operation. This prevents mutually exclusive operations like backup and restore to be executed simultaneously.

When the operation is complete, the `pbm-agent` releases the lock and updates the PBM control collections.

A single `pbm-agent` is involved with only one cluster (or non-sharded replica set). The `pbm` CLI utility can connect to any cluster to which it has network access, so it is possible for one user to list and launch backups or restores on many clusters.
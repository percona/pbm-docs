# PBM control collections

The config and state (current and historical) for backups is stored in
collections in the MongoDB cluster or non-sharded replica set itself. These are
put in the system `admin` db to keep them cleanly separated from user db namespaces.

In sharded clusters, this is the `admin` db of the config server replica set. In a non-sharded replica set, the PBM control collections are stored in
`admin` db of the replica set itself.

* *admin.pbmBackups* - Log / status of each backup.
* *admin.pbmAgents* - Contains information about `pbm-agents` statuses and health.
* *admin.pbmConfig* - Contains configuration information for Percona Backup for MongoDB.
* *admin.pbmCmd* - Is used to define and trigger operations.
* *admin.pbmLock* - **pbm-agent** synchronization-lock structure.
* *admin.pbmLockOp* - Is used to coordinate operations that are not mutually exclusive such as make backup and delete backup.
* *admin.pbmLog* - Stores log information from all `pbm-agents` in the MongoDB environment. Available in Percona Backup for MongoDB as of version 1.4.0.
* *admin.pbmOpLog* - Stores [operation IDs](../reference/glossary.md#opids).
* *admin.pbmPITRChunks* - Stores [Point-in-time recovery](../reference/glossary.md#point-in-time-recovery) oplog slices.
* *admin.pbmPITRState* - Contains current state of Point-in-time recovery incremental backups.
* *admin.pbmRestores* - Contains restore history and the restore state for all replica sets.
* *admin.pbmStatus* - Stores Percona Backup for MongoDB status records.

The `pbm` command line tool creates these collections as needed. You do not
have to maintain these collections, but you should not drop them unnecessarily
either. Dropping them during a backup will cause an abort of the backup.

Filling the config collection is a prerequisite to using Percona Backup for MongoDB for executing backups or restores. (See config page later.)
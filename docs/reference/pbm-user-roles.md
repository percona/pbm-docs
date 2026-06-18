# PBM user database roles

Percona Backup for MongoDB requires a dedicated database user holding a specific set of MongoDB roles. The table below gives a quick overview; the sections that follow explain why each role is needed and what it does — and does not — grant.

| Role            | Scope            | Access type  | Purpose                                       |
|-----------------|------------------|--------------|-----------------------------------------------|
| `readWrite`     | `admin` db only  | Read + write | PBM metadata / agent coordination collections |
| `backup`        | All databases    | Read-only    | Full data + oplog snapshot for backup         |
| `clusterMonitor`| Cluster metadata | Read-only    | Topology awareness, safe scheduling           |
| `restore`       | All databases    | Write        | Replay data during restore operations         |
| `pbmAnyAction`  | All resources    | Any action   | Privileged commands not covered by built-in roles (`fsync`, `setParameter`, etc.) |

## Built-in MongoDB roles

### `readWrite` (on `admin` db)

PBM stores its operational metadata — backup status, restore progress, configuration, and agent coordination — in the PBM control collections in the `admin` database (for example, `admin.pbmConfig`, `admin.pbmBackups`, `admin.pbmAgents`, and others; see [PBM control collections](../details/control-collections.md)). The `readWrite` role grants PBM the ability to read from and write to these collections so all agents in a cluster can share state and coordinate operations.
This role is scoped to the `admin` database only. It does not grant read or write access to application databases.

### `backup`

The `backup` role is a MongoDB built-in role that allows reading all data across all databases and collections (via `listDatabases`, `listCollections`, and `find` on all namespaces), as well as reading the oplog (`local.oplog.rs`). PBM requires this to stream a consistent snapshot of all data during a backup, including the oplog data needed for point-in-time recovery (PITR).

This is a read-only role with respect to application data. No write access to application databases is granted by this role alone.

### `clusterMonitor`

PBM agents need to query the cluster topology and health — member states, replication lag, shard configuration, and whether a node is primary or secondary — to make safe decisions about where to run a backup, when to wait for replication to catch up, and how to coordinate across a replica set or sharded cluster. The `clusterMonitor` role grants access to commands like `serverStatus`, `replSetGetStatus`, `listShards`, and similar diagnostic commands.

This role grants read-only access to monitoring and operational metadata. It does not allow modifying cluster configuration or accessing application data.

### `restore`

During a restore operation, PBM needs to write data back into all databases and collections, recreate indexes, and replay oplog entries to reach a consistent point-in-time state. The `restore` role grants the necessary `insert`, `update`, and `createCollection`/`createIndex` privileges across all namespaces to perform these operations.

This role is only exercised during a restore operation. It grants broad write privileges and should be treated as a sensitive, privileged role in your access review process.

## Custom role

### `pbmAnyAction` (custom role on `admin` db)

This custom role grants the `anyAction` privilege on `anyResource`, which is effectively full administrative access. PBM uses this capability for a limited set of commands that fall outside what the standard built-in roles cover:
- `setParameter` / `getParameter` — to read or temporarily adjust certain `mongod` parameters during backup/restore (for example, disabling the balancer on sharded clusters).
- `fsync` / `fsyncUnlock` — on some backup paths (particularly for physical/hot backups with Percona Server for MongoDB), PBM may need to flush and lock the WiredTiger storage engine.
- Oplog-related and internal commands — certain replication and storage-engine commands needed for consistent point-in-time backups are not exposed by any standard role.

!!! note "Compliance note"

    This is the most privileged role in the set. It is intentionally custom and narrowly justified by gaps in MongoDB's built-in role model. This role is assigned to a dedicated service account (`pbmuser`) used exclusively by the PBM agent process — not by any human operator or application.

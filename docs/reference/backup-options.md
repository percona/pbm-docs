# Backup options

```yaml
backup:
  priority:
    "localhost:28019": 2.5
    "localhost:27018": 2.5
    "localhost:27020": 2.0
    "localhost:27017": 0.1
  compression: <string>
  compressionLevel: <int>
  timeouts:
    startingStatus: 60
  oplogSpanMin: <float64>
  numParallelCollections: <int>
```

### priority

*Type*: array of strings

The list of `mongod` nodes and their priority for making backups. The node with the highest priority is elected for making a backup. If several nodes have the same priority, the one among them is randomly elected to make a backup.

If not set, the replica set nodes have the default priority as follows:

* hidden nodes - 2.0
* secondary nodes - 1.0
* primary node - 0.5

### backup.compression

*Type*: string <br>
*Default*: s2

The compression method for backup snapshots. Available in Percona Backup for MongoDB as of version 1.8.0.

When `none` is specified, backups are made without compression.

Supported values: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`. Default: `s2`.

<!-- backup-compression-level: -->
### backup.compressionLevel

*Type*: int

The compression level. The default value depends on the compression method used. 

The following table shows available compression levels per compression method:

| Compression method   | Supported compression levels | Default
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - fastest speed, 2 - default, 3 - better compression, 4 - best compression | 2
| `snappy`             | no levels|
| `lz4`                | From 1 (fastest) to 16 | 1
| `gzip` and `pgzip`   | -1 - default compression, 0 - no compression, 1 - best speed, 9 - best compression| -1

Note that the greater value you specify, the more time and computing resources it will take to compress the data.

### backup.timeouts.startingStatus

*Type*: unit32 <br>
*Default*: 33

The wait time (in seconds) for PBM to start backups. This timeout controls how long PBM waits for the backup to transition from initial state to running status.

  For physical backups in sharded clusters, this includes the time needed to open the `$backupCursor` on all shards. For logical backups, this includes gathering collection statistics
   (collStats operations) and creation of backup metadata.

  Increasing this value is useful when:
  - Physical backups take longer than usual to open the $backupCursor
  - Large databases with many collections require extended time for metadata collection
  - Slow storage systems delay backup initialization

  Applies to all backup types (logical, physical, incremental, external) on both single replica sets and sharded clusters.

  The 0 (zero) value resets the timeout to the default 33 seconds.

### backup.oplogSpanMin

*Type*: float64 <br>

The duration (in minutes) of oplog slices saved with the logical backup snapshot. By default, the duration of backup oplog slices equals to the value defined for the [`pitr.oplogSpanMin`](pitr-options.md#pitroplogspanmin) option (default - 10 minutes). You can reduce the duration in heavy-loaded environments. Note that setting the duration to shorter periods may increase the overall backup execution time. 

### backup.numParallelCollections

*Type*: int <br>
*Default*: number of CPU cores / 2

The number of parallel collections to process during a logical backup. By default, the number of parallel collections is half of the number of CPU cores. By setting the value for this option you define the new default.
Available starting with version 2.7.0.


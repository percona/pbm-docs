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

The wait time (in seconds) for PBM to start physical backups on all shards. Increasing this value is useful when it takes longer than usual to open the `$backupCursor`.

The 0 (zero) value resets the timeout to the default 33 seconds. 
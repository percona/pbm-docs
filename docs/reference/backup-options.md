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

The compression level from `0` till `10`. Default value depends on the compression method used.

Note that the higher value you specify, the more time and computing resources it will take to compress / retrieve the data.
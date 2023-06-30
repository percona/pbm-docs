# Point-in-time recovery options

```yaml
pitr:
  enabled: <boolean>
  oplogSpanMin: <float64>
  compression: <string>
  compressionLevel: <int>
```

### pitr.enabled

*Type*: boolean <br>
*Default*: False

Enables point-in-time recovery

### pitr.oplogSpanMin

*Type*: float64 <br>
*Default*: 10

The duration of an oplog span in minutes. If set when the `pbm-agent` is making an oplog slice, the slice’s span is updated right away.

If the new duration is smaller than the previous one, the `pbm-agent` is triggered to save a new slice with the updated span. If the duration is larger, then the next slice is saved with the updated span in scheduled time.

### pitr.compression

*Type*: string <br>
*Default*: s2

The compression method for Point-in-Time Recovery oplog slices. Available in Percona Backup for MongoDB as of version 1.7.0.

Supported values: `gzip`, `snappy`, `lz4`, `s2`, `pgzip`, `zstd`. Default: `s2`.

### pitr.compressionLevel

*Type*: int

The compression level is from `0` till `10`. The default value depends on the compression method used.

The following table shows available compression levels per compression method:

| Compression method   | Compression levels           | Default
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - fastest speed, 2 - default, 3 - better compression, 4 - best compression | 2
| `snappy`             | no levels|
| `lz4`                | From 1 (fastest) to 16 | 1
| `gzip` and `pgzip`   | -1 - default compression, 0 - no compression, 1 - best speed, 9 - best compression| -1


Note that the greater value you specify, the more time and computing resources it will take to compress the data.

### pitr.oplogOnly

*Type*: boolean <br>
*Default*: False <br>
*Required*: NO

Controls whether the base backup is required to start [Point-in-Time recovery oplog slicing](../features/point-in-time-recovery.md#oplog-slicing). When set to true, Percona Backup for MongoDB saves oplog chunks without the base backup snapshot.

Available in Percona Backup for MongoDB starting with version 1.8.0. To learn more about the usage, see [Point-in-Time Recovery oplog replay](../usage/oplog-replay.md).
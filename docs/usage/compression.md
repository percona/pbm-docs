# Configure backup compression

By default, Percona Backup for MongoDB uses the `s2` compression method when making a backup.

You can start a backup with a different compression method by passing the `--compression` flag to the **pbm backup** command.

For example, to start a backup with `gzip` compression, use the following command:

```{.bash data-prompt="$"}
$ pbm backup --compression=gzip
```

Supported compression types are: `gzip`, `snappy`, `lz4`, `pgzip`, `zstd`.  The `none` value means no compression is done during backup.

As of version 1.7.0, you can configure the compression level for backups. Specify the value for the [`--compression-level`](../reference/backup-options.md#backupcompressionlevel) flag. 

Default compression levels differ per compression method used. 

The following table shows available compression levels per compression method:

| Compression method   | Supported compression levels | Default
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - fastest speed, 2 - default, 3 - better compression, 4 - best compression | 2
| `snappy`             | no levels|
| `lz4`                | From 1 (fastest) to 16 | 1
| `gzip` and `pgzip`   | -1 - default compression, 0 - no compression, 1 - best speed, 9 - best compression| -1

Note that the higher value you specify, the more time and computing resources it will take to compress the data.
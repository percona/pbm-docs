# Percona Backup for MongoDB diagnostics tools

Percona Backup for MongoDB provides diagnostics tools to operate data backups.

## pbm-speed-test

**pbm-speed-test** allows field-testing compression and backup upload speed of logical backups. You can use it:

* To check performance before starting a backup

* To find out what slows down the running backup

By default, **pbm-speed-test** operates with fake semi random data documents. To
run **pbm-speed-test** on a real collection, provide a valid MongoDB connection URI string for the `--mongodb-uri` flag.

Run **pbm-speed-test** for the full set of available commands.

### Compression test

```bash
pbm-speed-test compression --compression=s2 --size-gb 10
```

??? example "Sample output"

    ```{.bash .no-copy}
    Test started ....
    10.00GB sent in 8s.
    Avg upload rate = 1217.13MB/s.
    ```

**pbm-speed-test compression** uses the compression library from the config
file and sends a fake semi random data document (1 GB by default) to the
black hole storage. (Use the `pbm config` command to change the compression library).

To test compression on a real collection, pass the
`--sample-collection` flag with the `<my_db.my_collection>` value.

Run `pbm-speed-test compression --help` for the full set of supported flags:

```bash
pbm-speed-test compression --help
```

??? example "Sample output"

    ```{.text .no-copy}
    usage: pbm-speed-test compression

    Run compression test

    Flags:
          --help                     Show context-sensitive help (also try
                                     --help-long and --help-man).
          --mongodb-uri=MONGODB-URI  MongoDB connection string
      -c, --sample-collection=SAMPLE-COLLECTION
                                     Set collection as the data source
      -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
          --compression=s2           Compression type
                                     <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
          --compression-level=COMPRESSION-LEVEL
                                     Compression level (specific to the compression type)
                                     <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>

### Upload speed test

```bash
pbm-speed-test storage --compression=s2
```

??? example "Sample output"

    ```
    Test started
    1.00GB sent in 1s.
    Avg upload rate = 1744.43MB/s.
    ```

`pbm-speed-test storage` sends the semi random data (1 GB by default) to the
remote storage defined in the config file. Pass the `--size-gb` flag to change the
data size.

To run the test with the real collection’s data instead of the semi random data,
pass the `--sample-collection` flag with the `<my_db.my_collection>` value.

Run `pbm-speed-test storage --help` for the full set of available flags:

```bash
pbm-speed-test storage --help
```

??? example "Sample output"

    ```{.text .no-copy}
    usage: pbm-speed-test storage

    Run storage test

    Flags:
          --help                     Show context-sensitive help (also try --help-long and --help-man).
          --mongodb-uri=MONGODB-URI  MongoDB connection string
      -c, --sample-collection=SAMPLE-COLLECTION
                                     Set collection as the data source
      -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
          --compression=s2           Compression type <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
          --compression-level=COMPRESSION-LEVEL
                                    Compression level (specific to the compression type)



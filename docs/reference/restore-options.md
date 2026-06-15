# Restore options

```yaml
restore:
  batchSize: <int>
  numInsertionWorkers: <int>
  numParallelCollections: <int>
  numDownloadWorkers: <int>
  maxDownloadBufferMb: <int>
  downloadChunkMb: <int>
  mongodLocation: <string>
  mongodLocationMap:
     "node01:2017": <string>
     "node03:27017": <string>
```

### restore.batchSize

*Type*: int <br>
*Default*: 500

The number of documents to buffer.

### restore.numInsertionWorkers

*Type*: int <br>
*Default*: 10

Specifies the number of insertion workers to run concurrently per collection. 

### restore.numParallelCollections

*Type*: int <br>
*Default*: number of CPU cores / 2

The number of collections to process in parallel during a logical restore. The default value is half of the number of CPU cores. By setting the value for this option you define the new default.
Available starting with version 2.7.0.

### restore.numDownloadWorkers

*Type*: int <br>
*Default*: number of CPU cores

The number of workers that request data chunks from the storage during the restore. The default value equals to the number of CPU cores.

### restore.maxDownloadBufferMb

*Type*: int <br>
*Default*: `numDownloadWorkers * downloadChunkMb * 16` MB when unspecified or set to `0`

The maximum size of the in-memory buffer that is used to download files from the S3 storage. When unspecified or set to 0, the size cannot exceed the value calculated as `numDownloadWorkers * downloadChunkMb * 16` MB. By default, the number of CPU cores * 32 * 16 MB.

### restore.downloadChunkMb

*Type*: int <br>
*Default*: 32

The size of the data chunk in MB to download from the S3 storage.

### restore.mongodLocation

*Type*: string


The custom path to `mongod` binaries. When undefined, Percona Backup for MongoDB uses the default path to start the temporary instances required during physical restore. After a physical restore the database is not started automatically.

### restore.mongodLocationMap

*Type*: array of strings

The list of custom paths to `mongod` binaries on every node. Percona Backup for MongoDB uses the values to start the temporary instances required during physical restore. After a physical restore the database is not started automatically.

### restore.timeouts.balancerStop

*Type*: int <br>
*Default*: 0 (unlimited)

Defines the maximum time (in seconds) that PBM waits for the balancer to stop before starting a logical restore. If set to `0` or not set (default), PBM waits indefinitely until the balancer stops.

PBM stops the balancer to ensure data consistency during restore operations in sharded clusters. The timeout starts when the stop request is issued. If the balancer remains active after the timeout, the restore is aborted.

```yaml
restore:
  timeouts:
    balancerStop: 0
```

??? example "Example"
    ```yaml
    restore:
      timeouts:
        balancerStop: 60
    ```

    In this example, PBM waits up to 60 seconds for the balancer to stop. If the balancer is still running after this period, the restore fails.


This is useful when you want to:

- Prevent restore operations from waiting indefinitely
- Enforce time limits in automated workflows
- Fail fast if the balancer cannot be stopped

### restore.indexCommitQuorum

*Type*: string or int <br>
*Default*: `votingMembers`

Specifies how many data-bearing voting nodes must complete an index build before the primary node commits the index during a **logical restore** operation.

By default, Percona Backup for MongoDB waits for all voting members (`votingMembers`) to complete index building. In large replica sets, this can introduce significant delays if some nodes build indexes slower than others, blocking the entire restore process.

Following MongoDB index build commit quorum semantics, you can optimize restore performance by setting this option to a lower quorum threshold.

The following values are supported:

`votingMembers` (Default): The primary waits for all data-bearing voting members to complete the index build.

`majority`: The primary commits the index as soon as a simple majority of data-bearing voting members have completed the build. This is recommended for large replica sets to prevent lagging nodes from delaying the restore process.

`<int>`: A specific number of data-bearing voting members (e.g., 3) that must complete the index build. The integer value must be between 1 and the number of data-bearing voting members. A value of 1 means the primary commits immediately after it finishes.

```sh
pbm config --set restore.indexCommitQuorum=majority
```

??? example "Example"

    1. Run the following command:

        ```sh
        pbm config --set restore.indexCommitQuorum=majority
        ```

    2. Confirm the value was set:

        ```sh
        pbm config restore.indexCommitQuorum=
        majority
        [restore.indexCommitQuorum=majority]
        ```

    3. Confirm `indexCommitQuorum` appears in the PBM config with the following command:

        ```text
        $ pbm config
        storage:
          type: s3
          s3:
            region: us-east-1
            endpointUrl: http://minio:9000
            forcePathStyle: true
            bucket: bcp
            prefix: pbme2etest
            credentials:
              access-key-id: '***'
              secret-access-key: '***'
            maxUploadParts: 10000
            storageClass: STANDARD
            insecureSkipTLSVerify: false
        pitr:
          enabled: false
          compression: s2
        backup:
          oplogSpanMin: 0
          compression: s2
        restore:
          indexCommitQuorum: majority
        ```

    4. Start the restore and wait for it to complete:

        ```sh
        pbm restore -w 2026-05-12T13:28:07Z
        ```
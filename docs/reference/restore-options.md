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

## restore.batchSize

*Type*: int <br>
*Default*: 500

The number of documents to buffer.

## restore.numInsertionWorkers

*Type*: int <br>
*Default*: 10

Specifies the number of insertion workers to run concurrently per collection. 

## restore.numParallelCollections

*Type*: int <br>
*Default*: number of CPU cores / 2

The number of collections to process in parallel during a logical restore. The default value is half of the number of CPU cores. By setting the value for this option you define the new default.
Available starting with version 2.7.0.

## restore.numDownloadWorkers

*Type*: int <br>
*Default*: number of CPU cores

The number of workers that request data chunks from the storage during the restore. The default value equals to the number of CPU cores.

## restore.maxDownloadBufferMb

*Type*: int <br>
 

The maximum size of the in-memory buffer that is used to download files from the S3 storage. When unspecified or set to 0, the size cannot exceed the value calculated as `numDownloadWorkers * downloadChunkMb * 16` MB. By default, the number of CPU cores * 32 * 16 MB.

## restore.downloadChunkMb

*Type*: int <br>
*Default*: 32

The size of the data chunk in MB to download from the S3 storage.

## restore.mongodLocation

*Type*: string


The custom path to `mongod` binaries. When undefined, Percona Backup for MongoDB uses the default path to start the temporary instances required during physical restore. After a physical restore the database is not started automatically.

## restore.mongodLocationMap

*Type*: array of strings

The list of custom paths to `mongod` binaries on every node. Percona Backup for MongoDB uses the values to start the temporary instances required during physical restore. After a physical restore the database is not started automatically.

## restore.timeouts.balancerStop

*Type*: int <br>
*Default*: 0

Defines the maximum time (in seconds) that Percona Backup for MongoDB (PBM) waits for the balancer to stop before starting a logical restore.

PBM stops the balancer to ensure data consistency during restore operations in sharded clusters. The timeout starts when the stop request is issued. If the balancer remains active after the timeout, the restore is aborted.

```yaml
restore:
  timeouts:
    balancerStop: 0
```
- 0 (default): Wait indefinitely until the balancer stops
- `> 0`: Maximum time (in seconds) to wait before failing the restore

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
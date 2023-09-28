# Restore options

```yaml
restore:
  batchSize: <int>
  numInsertionWorkers: <int>
  numDownloadWorkers: <int>
  maxDownloadBufferMb: <int>
  downloadChunkMb: <int>
  mongodLocation: <string>
  mongodLocationMap:
     "node01:2017": <string>
     "node03:27017": <string>
```

### batchSize

*Type*: int <br>
*Default*: 500

The number of documents to buffer.

### numInsertionWorkers

*Type*: int <br>
*Default*: 10

The number of workers that add the documents to buffer.

### numDownloadWorkers

*Type*: int <br>
*Default*: number of CPU cores

The number of workers that request data chunks from the storage during the restore. The default value equals to the number of CPU cores.

### maxDownloadBufferMb

*Type*: int <br>
 

The maximum size of the in-memory buffer that is used to download files from the S3 storage. When unspecified or set to 0, the size cannot exceed the value calculated as `numDownloadWorkers * downloadChunkMb * 16` MB. By default, the number of CPU cores * 32 * 16 MB.

### downloadChunkMb

*Type*: int <br>
*Default*: 32

The size of the data chunk in MB to download from the S3 storage.

### mongodLocation

*Type*: string

The custom path to `mongod` binaries. When undefined, Percona Backup for MongoDB uses the default path to make database restarts during physical restore.

### mongodLocationMap

*Type*: array of strings

The list of custom paths to `mongod` binaries on every node. Percona Backup for MongoDB uses the values to make restarts of the database during physical restore. 
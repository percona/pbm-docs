# Restore options

```yaml
restore:
  batchSize: <int>
  numInsertionWorkers: <int>
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

### mongodLocation

*Type*: string

The custom path to `mongod` binaries. When undefined, Percona Backup for MongoDB uses the default path to make database restarts during physical restore.

### mongodLocationMap

*Type*: array of strings

The list of custom paths to `mongod` binaries on every node. Percona Backup for MongoDB uses the values to make restarts of the database during physical restore. 
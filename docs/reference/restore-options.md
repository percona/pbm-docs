# Restore options

```yaml
restore:
  batchSize: <int>
  numInsertionWorkers: <int>
```

### batchSize

*Type*: int <br>
*Default*: 500

The number of documents to buffer.

### numInsertionWorkers

*Type*: int <br>
*Default*: 10

The number of workers that add the documents to buffer.
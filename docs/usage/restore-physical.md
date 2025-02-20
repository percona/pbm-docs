# Restore from a physical backup

--8<-- "restore-intro.md"

## Considerations

1. Disable point-in-time recovery. A restore and point-in-time recovery oplog slicing are incompatible operations and cannot be run simultaneously. 

    ```{.bash data-prompt="$"}
    $ pbm config --set pitr.enabled=false
    ```

2. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
3. Make sure all nodes in the cluster are healthy (i.e. either PRIMARY or SECONDARY). Each pbm-agent needs to be able to connect to its local node and run queries in order to perform the restore.
4. For PBM versions before 2.1.0, physical restores are not supported for deployments with arbiter nodes.


## Before you start

1. Shut down all `mongos` nodes as the database won't be available while the restore is in progress. 
2. Shut down all `pmm-agent` and other clients that can do the write operations to the database. This is required to ensure data consistency after the restore.
3. Stop the arbiter nodes manually since there's no `pbm-agent` on these nodes to do that automatically.
   
## Restore a database

1. List the backups 

    ```{.bash data-prompt="$"}
    $ pbm list
    ```

2. Make a restore

    ```{.bash data-prompt="$"}
    $ pbm restore <backup_name>
    ```

    During the physical restore, `pbm-agent` processes stop the `mongod` nodes, clean up the data directory and copy the data from the storage onto every node. During this process, the database is restarted several times. 

    You can [track the restore progress](restore-progress.md) using the `pbm describe-restore` command. Don't run any other commands since they may interrupt the restore flow and cause the issues with the database.

### Post-restore steps

After the restore is complete, do the following:

1. Remove the contents of the datadir on any arbiter nodes
2. Restart all `mongod` nodes

    !!! note

        You may see the following message in the `mongod` logs after the cluster restart:

        ```{.text .no-copy}
        "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
        ```

        This is expected behavior of periodic checks upon the database start. During the restore, the `config.system.sessions` collection is dropped but Percona Server for MongoDB recreates it eventually. It is a normal procedure. No action is required from your end.

3. Restart all `pbm-agents`

4. Run the following command to resync the backup list with the storage:

    ```{.bash data-prompt="$"}
    $ pbm config --force-resync
    ``` 

5. Start the balancer and start `mongos` nodes.

6. We recommend to make a fresh backup to serve as the new base for future restores.
7. [Enable point-in-time recovery](../features/point-in-time-recovery.md#enable-point-in-time-recovery) if required.
     

## Define a `mongod` binary location

!!! admonition "Version added: 2.0.4"

During physical restores, Percona Backup for MongoDB performs several restarts of the database. By default, it uses the location of the `mongod` binaries from the `$PATH` variable to access the database. If you have defined the custom path to the `mongod` binaries, make Percona Backup for MongoDB aware of it by specifying this path in the configuration file: 

```yaml
restore:
    mongodLocation: /path/to/mongod
```

If you have different paths to `mongod` binaries on every node of your cluster / replica set, use the `mongodLocationMap` option to specify your custom paths for each node.

```yaml
restore:
    mongodLocationMap:
       "node01:27017": /path/to/mongod
       "node03:27017": /another/path/to/mongod
```

When running in Docker, include Percona Backup for MongoDB files together with your MongoDB binaries. See [Run Percona Backup for MongoDB in a Docker container](https://docs.percona.com/percona-backup-mongodb/install/docker.html) for more information.

### Parallel data download

!!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

Percona Backup for MongoDB downloads data chunks from the S3 storage concurrently during physical restore. Read more about benchmarking results in the [Speeding up MongoDB restores in PBM](https://www.percona.com/blog/speeding-up-database-restores-in-pbm) blog post by *Andrew Pogrebnoi*.

Here's how it works:

During the physical restore, Percona Backup for MongoDB starts the workers. The number of workers equals to the number of CPU cores by default. Each worker has a memory buffer allocated for it. The buffer is split into spans for the size of the data chunk. The worker acquires the span to download a data chunk and stores it into the buffer. When the buffer is full, the worker waits for the free span to continue the download.   

You can fine-tune the parallel download depending on your hardware resources and database load. Edit the PBM configuration file and specify the following settings:

```yaml
restore:
   numDownloadWorkers: <int>
   maxDownloadBufferMb: <int>
   downloadChunkMb: 32
```

* `numDownloadWorkers` - the number of workers to download data from the storage. By default, it equals to the number of CPU cores
* `maxDownloadBufferMb` - the maximum size of memory buffer to store the downloaded data chunks for decompression and ordering. It is calculated as `numDownloadWorkers * downloadChunkMb * 16`
* `downloadChunkMb` is the size of the data chunk to download (by default, 32 MB)


## Next steps

[Point-in-time recovery](../usage/pitr-physical.md){.md-button}

## Useful links 

* [View restore progress](../usage/restore-progress.md)
* [Restore into a new environment](../features/restore-new-env.md)
* [Restore into a cluster under a different name](../features/restore-remapping.md)



  




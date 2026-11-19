> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Filesystem storage

## Remote filesystem server storage

A remote filesystem server storage is a file server mounted to a local directory on each MongoDB node. The server administrators must ensure that all nodes in the cluster or replica set have the same remote directory mounted at exactly the same local path.

Warning

Percona Backup for MongoDB treats the directory as a normal local directory and does not verify that it is actually mounted from a remote server.

If you accidentally use a local directory instead of a remote mount, other nodes will not be able to access backup files. This will likely cause errors during restore operations, as **pbm-agent** processes on other replica set members cannot access backup archives stored on a local-only directory.

### Configuration example

You can find [the configuration file template](https://github.com/percona/percona-backup-mongodb/blob/v2.17.0/packaging/conf/pbm-conf-reference.yml) and uncomment the required fields.

```
storage:
  type: filesystem
  filesystem:
    path: /data/local_backups
```

For the description of configuration options, see [Configuration file options](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/configuration-options.html).

## Local filesystem storage

Local filesystem storage is only supported for single-node replica sets. Using it in a multi-node setup is not recommended, as other nodes will not have access to the backup files.

For testing, you can use any object store you are familiar with. If you do not have an object store, we recommend MinIO for its simple setup. If you plan to use a remote filesystem-type backup server, see the [Remote filesystem server storage](#remote-filesystem-server-storage) section above.

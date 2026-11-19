> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Architecture

Percona Backup for MongoDB consists of the following components:

- [`pbm-agent`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#pbm-agent) is a process running on **every** `mongod` node that is not an arbiter node within the cluster or within a replica set that performs backup and restore operations.
- [`pbm` CLI](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#pbm-cli) is a command-line utility that instructs `pbm-agents` to perform an operation.
- [PBM Control collections](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#pbm-control-collections) are special collections in MongoDB that store the configuration data and backup states. Both `pbm` CLI and `pbm-agent` use PBM Control collections to check backup status in MongoDB and communicate with each other.
- Remote backup storage is where Percona Backup for MongoDB saves backups. It can be either [an S3 compatible storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#s3-compatible-storage) or a filesystem-type storage.

The following diagram illustrates how Percona Backup for MongoDB components communicate with MongoDB.

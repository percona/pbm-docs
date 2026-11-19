> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Comparison with MongoDB

Percona Backup for MongoDB is a fully supported community backup solution that can perform cluster-wide consistent backups in MongoDB. The following table compares Percona Backup for MongoDB with the MongoDB backup solutions:

| Feature name                                   | Percona Backup for MongoDB                                          | MongoDB Community `mongodump`      | MongoDB Enterprise                                       | MongoDB Atlas                                |
| ---------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------- | -------------------------------------------------------- | -------------------------------------------- |
| Open source software                           | Yes                                                                 | No                                 | No                                                       | No                                           |
| Sharded cluster backups and restores supported | Yes                                                                 | No                                 | Yes                                                      | Yes                                          |
| Binary database export (logical backup)        | Yes                                                                 | Yes                                | Yes                                                      | Yes                                          |
| Selective backup (logical)                     | Yes                                                                 | Yes                                | Yes                                                      | Yes                                          |
| Built-in point-in-time recovery support        | Yes                                                                 | No                                 | Yes                                                      | Yes                                          |
| Physical backup                                | Yes                                                                 | No                                 | Yes                                                      | Yes                                          |
| Incremental backup (physical)                  | Yes                                                                 | No                                 | Yes                                                      | Yes                                          |
| Backup management interfaces                   | Percona Backup for MongoDB (CLI) PMM mongodump / mongorestore (CLI) | - - mongodump / mongorestore (CLI) | Ops Manager Cloud Manager mongodump / mongorestore (CLI) | Atlas backups mongodump / mongorestore (CLI) |
| Multiple storages for backups                  | Yes                                                                 | No                                 | No                                                       | No                                           |

## What you get with Percona Backup for MongoDB

- [Enterprise features without extra costs](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/comparison.html)
- [Works for both sharded clusters and non-sharded replica sets](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/deployments.html)
- [Simple command-line management utility](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html). For backup management via a user interface, consider [using PBM through Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html)
- Simple, [integrated-with-MongoDB authentication](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/authentication.html#external-authentication-support-in-percona-backup-for-mongodb)
- Distributed transaction consistency with MongoDB 4.2+
- Compatibility with different storage types: [S3-compatible storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/s3-storage.html), [Microsoft Azure Blob storage](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/azure.html), `filesystem` storage type for [locally-mounted remote filesystem backup servers](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/filesystem-storage.html#remote-filesystem-server-storage)

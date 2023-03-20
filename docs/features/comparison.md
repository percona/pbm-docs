# Comparison with MongoDB 

Percona Backup for MongoDB is a fully supported community backup solution that can perform cluster-wide consistent backups in MongoDB. The following table compares Percona Backup for MongoDB with the MongoDB backup solutions:

| Feature name | Percona Backup for MongoDB | MongoDB Community `mongodump` | MongoDB Enterprise | MongoDB Atlas |
| -------------| -------------------------- | ----------- | ----------------- | --------------- 
|Open source backup | Yes | No	| No | No 
| Binary database export (logical backup) | Yes | Yes | Yes | Yes 
| Built-in point-in-time recovery support |	Yes | No | Yes | Yes
| Physical backup |	Yes	| No | Yes | Yes
| Incremental backup (physical) | Yes |	No |Yes | Yes
| Backup management interfaces	| Percona Backup for MongoDB (CLI) <br> PMM <br> mongodump / mongorestore (CLI) | - <br> - <br> mongodump / mongorestore (CLI) | Ops Manager <br> Cloud Manager <br> mongodump / mongorestore (CLI) | Atlas backups <br> mongodump / mongorestore (CLI)
| Sharded cluster restores supported | Yes | No | Yes |	Yes

## What you get with Percona Backup for MongoDB

* [Enterprise features without extra costs](comparison.md) 
* [Works for both sharded clusters and non-sharded replica sets](../details/deployments.md)
* [Simple command-line management utility](../reference/pbm-commands.md). For backup management via a user interface, consider [using PBM through Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html)
* Simple, [integrated-with-MongoDB authentication](../install/initial-setup.md#external-authentication-support-in-percona-backup-for-mongodb)
* Distributed transaction consistency with MongoDB 4.2+
* Compatibility with different storage types: [S3-compatible storage](../details/storage-configuration.md#s3-compatible-storage), [Microsoft Azure Blob storage](../details/storage-configuration.md#microsoft-azure-blob-storage), `filesystem` storage type for [locally-mounted remote filesystem backup servers](../details/storage-configuration.md#remote-filesystem-server-storage)
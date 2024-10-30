# Supported MongoDB versions

Percona Backup for MongoDB is compatible with:

* MongoDB Community / Enterprise Edition with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) enabled. Percona Backup for MongoDB supports only *logical backups*.
* Percona Server for MongoDB with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) for logical backups. For physical and incremental backups, also configure WiredTiger as the storage engine.

Navigate to the required compatibility matrix to see what MongoDB versions your current PBM release is compatible with for backups and restores. 

!!! note ""

    End-of-life MongoDB versions may work with PBM, but they are not tested for compatibility. Consider using a previous PBM version in this case.

## Percona Server for MongoDB compatibility matrix

The following table lists the Percona Server for MongoDB versions supported for each backup type.

Each entry indicates the PBM version that introduces changes in the supported Percona Server for MongoDB versions.

| PBM version | Logical | Physical |Incremental physical | Restore backward compatibility|
| ----------- |---------|----------|---------------------|---------------------|
| **2.4.0 - 2.7.0** | 5.0.x, 6.0.x, 7.0.x | 5.0.x, 6.0.x, 7.0.x | 5.0.14-12, 6.0.3-2 and higher, 7.0.x | Yes |
| **2.2.0 - 2.3.0** | 4.4 and higher| 4.4.6-8 and higher, 5.0.x, 6.0.x| 4.4.18, 5.0.14-12, 6.0.3-2 and higher| Yes |
| **2.1.0** | 4.4 and higher | 4.4.6-8, 5.0.x, 6.0.x| 4.2.24-24, 4.4.18, 5.0.14-12, 6.0.3-2 and higher| No. A fresh backup is required|
| **1.7.0** | 4.2 and higher| tech preview: 4.2.15-16, 4.4.6-8, 5.0 and higher| | Yes
| **1.6.1** | 3.6 and higher | N/A |N/A |N/A |Yes


## MongoDB Community / Enterprise Edition compatibility matrix

This table lists the supported MongoDB Community and Enterprise Edition versions for logical backups. Each entry indicates the new PBM version that introduces changes in the supported MongoDB versions. 

| PBM version | Logical backups | Restore backward compatibility|
| ----------- |-----------------| ----------------------------- |
| **2.4.0 - 2.7.0** | 5.0.x and higher | Yes |
| **2.2.0 - 2.3.0** | 4.4.x and higher | Yes |
| **2.1.0** | 4.4.x and higher| No. A fresh backup is required|
| **1.7.0** | 4.2 and higher| Yes
| **1.6.1** | 3.6 and higher|Yes





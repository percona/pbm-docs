# Supported MongoDB versions

Percona Backup for MongoDB is compatible with:

* MongoDB Community / Enterprise Edition enabled with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/). Percona Backup for MongoDB supports only *logical backups*.
* Percona Server for MongoDB for all [supported backup types](../features/backup-types.md). Make sure [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) is enabled for logical backups. Physical and incremental backup works only with the WiredTiger storage engine.

Please take a look at the compatibility matrices below to check which MongoDB versions your current PBM release supports for backups and restores. 

!!! note ""

    End-of-life MongoDB versions may work with PBM but are not tested for compatibility. Consider upgrading to the newer MongoDB or Percona Server for the MongoDB version. 

## Percona Server for MongoDB compatibility matrix

The following table lists the Percona Server for MongoDB versions supported for each backup type.

Each entry indicates the PBM version that introduces changes in the supported Percona Server for MongoDB versions.

The Restore compatibility column indicates if you can restore from the backup made with the previous version of PBM.

| PBM version | Logical | Physical |Incremental physical | Restore backward compatibility|
| ----------- |---------|----------|---------------------|---------------------|
| **2.6.0 - 2.7.0** | [5.0.x] - [7.0.x] | [5.0.x] - [7.0.x] | [5.0.14-12]+, [6.0.3-2]+, [7.0.x] | Yes |
| **2.3.1 - 2.5.0** | 4.4.x - [7.0.x] | 4.4.6-8+ - [7.0.x] | 4.4.18-18+, [5.0.14-12], [6.0.3-2]+, [7.0.x] | Yes |
| **2.3.0** | 4.4.x - [6.0.x], | 4.4.6-8+, 5.0.x, 6.0.x | 4.4.18-18+, [5.0.14-12]+, [6.0.3-2]+ | Yes |
| **2.1.0 - 2.2.1** | 4.2.x - [6.0.x] | 4.4.6-8+, [5.0.x], [6.0.x] | 4.2.24-24+, 4.4.18-18+, [5.0.14-12]+, [6.0.3-2]+ | No. A fresh backup is required|
| **2.0.0** | 4.2 - [5.0.x] | 4.2.15-16+, 4.4.16-16+, [5.0.x] | N/A | Yes|
| **1.7.0 - 1.8.1** | 4.2 - [5.0.x] | tech preview: 4.2.15-16, 4.4.6-8, [5.0.x] | N/A | Yes|
| **1.6.1** | 4.0 - [5.0.x] | N/A |N/A |N/A |Yes|
| **1.6.0** | 3.6 - [5.0.x] | N/A |N/A |N/A |Yes|
| **1.0.0 - 1.5.0** | 3.6 - 4.4 | N/A |N/A |N/A |Yes|


## MongoDB Community / Enterprise Edition compatibility matrix

This table lists the supported MongoDB Community and Enterprise Edition versions for logical backups. Each entry indicates the new PBM version that introduces changes in the supported MongoDB versions. 

The Restore compatibility column indicates if you can restore from the backup made with the previous version of PBM.

| PBM version | Logical backups | Restore backward compatibility|
| ----------- |-----------------| ----------------------------- |
| **2.6.0 - 2.7.0** | 5.0.x and higher | Yes |
| **2.2.0 - 2.5.0** | 4.4.x and higher | Yes |
| **2.1.0** | 4.4.x and higher| No. A fresh backup is required|
| **1.7.0** | 4.2 and higher| Yes
| **1.6.1** | 3.6 and higher|Yes



[7.0.x]: https://docs.percona.com/percona-server-for-mongodb/7.0/
[6.0.x]: https://docs.percona.com/percona-server-for-mongodb/6.0/
[6.0.3-2]: https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.3-2.html
[5.0.x]: https://docs.percona.com/percona-server-for-mongodb/5.0/
[5.0.14-12]: https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.14-12.html
[4.4.x]: https://docs.percona.com/percona-server-for-mongodb/4.4/
[4.4.18-18]: https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.18-18.html
[4.4.6-8]: https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.6-8.html
[4.2.24-24]: https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.24-24.html
[4.2.15-16]: https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.15-16.html


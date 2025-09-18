# Supported MongoDB versions

Percona Backup for MongoDB is compatible with:

* MongoDB Community / Enterprise Edition with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) enabled. Percona Backup for MongoDB supports only *logical backups*.
* Percona Server for MongoDB for all [supported backup types](../features/backup-types.md). For logical backups, make sure [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) is enabled. For physical and incremental backups, make sure WiredTiger is set as the storage engine.

Refer to the compatibility matrices below to check which MongoDB versions your current PBM release supports for backups and restores. 

!!! note ""

    End-of-life MongoDB versions may work with PBM, but they are not tested for compatibility. Consider upgrading to the newer MongoDB or Percona Server for MongoDB version. 

## Percona Server for MongoDB compatibility matrix

The following table lists the Percona Server for MongoDB versions supported for each backup type.

Each entry indicates the PBM version that introduces changes in the supported Percona Server for MongoDB versions.

The Restore compatibility column indicates if you can restore from the backup made with the previous version of PBM.

| PBM version | Logical | Physical |Incremental physical | Restore backward compatibility|
| ----------- |---------|----------|---------------------|---------------------|
| **2.11.0** | [7.0.x], [8.0.x] | [7.0.x], [8.0.x] | [7.0.x], [8.0.x] | Yes |
| **2.8.0 - 2.10.0**  | [6.0.x], [7.0.x], [8.0.x] | [6.0.x], [7.0.x], [8.0.x] | [6.0.x], [7.0.x], [8.0.x] | Yes |
| **2.6.0 - 2.7.0** | [5.0.x], [6.0.x], [7.0.x] | [5.0.x], [6.0.x], [7.0.x] | [5.0.14-12], [6.0.3-2] and higher, [7.0.x] | Yes |
| **2.2.0 - 2.5.0** | [4.4.x] and higher| [4.4.6-8] and higher, 5.0.x, 6.0.x| [4.4.18-18], [5.0.14-12], [6.0.3-2] and higher| Yes |
| **2.1.0** | 4.4.x and higher | [4.4.6-8], [5.0.x], [6.0.x]| [4.2.24-24], [4.4.18-18], [5.0.14-12], [6.0.3-2] and higher| No. A fresh backup is required|
| **1.7.0** | 4.2 and higher| tech preview: [4.2.15-16], [4.4.6-8], 5.0 and higher| | Yes
| **1.6.1** | 3.6 and higher | N/A |N/A |N/A |Yes

## MongoDB Community / Enterprise Edition compatibility matrix

This table lists the supported MongoDB Community and Enterprise Edition versions for logical backups. Each entry indicates the new PBM version that introduces changes in the supported MongoDB versions. 

The Restore compatibility column indicates if you can restore from the backup made with the previous version of PBM.

| PBM version | Logical backups | Restore backward compatibility|
| ----------- |-----------------| ----------------------------- |
| **2.11.0** | 7.0.x and higher | Yes |
| **2.8.0 - 2.10.0**  | 6.0.x and higher | Yes |
| **2.6.0 - 2.7.0** | 5.0.x and higher | Yes |
| **2.2.0 - 2.5.0** | 4.4.x and higher | Yes |
| **2.1.0** | 4.4.x and higher| No. A fresh backup is required|
| **1.7.0** | 4.2 and higher| Yes
| **1.6.1** | 3.6 and higher|Yes




[8.0.x]: https://docs.percona.com/percona-server-for-mongodb/8.0/
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

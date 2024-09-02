# Supported MongoDB versions

Percona Backup for MongoDB is compatible with:

* MongoDB Community / Enterprise Edition  with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) enabled for logical backups.
* Percona Server for MongoDB with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) for logical backups. For physical backups, also configure WiredTiger as the storage engine.

The following table provides information about the supported MongoDB versions for each PBM release. 

!!! note ""

    End-of-life MongoDB versions may work with PBM, but they are not tested for compatibility. Consider using an previous PBM version in this case.


| PBM version | MongoDB Community / Enterprise | Percona Server for MongoDB|Compatibility with previous PBM versions|
| ----------- |------------------------------- | ------------------------- |----------------------------------------|
| [2.6.0](../release-notes/2.6.0.md) | <ul><li>Logical backups: version 5.0.x and higher</li></ul> | <ul><li>Logical backups: version 5.0.x and higher</li><li>Physical backups: version 5.0.x, 6.0.x, 7.0.x</li><li>Incremental backups: version 5.0.14-12, 6.0.3-2 and higher, 7.0.x</li></ul> | Yes |
| [2.5.0](../release-notes/2.5.0.md) | <ul><li>Logical backups: version 5.0.x and higher</li></ul> | <ul><li>Logical backups: version 5.0.x and higher</li><li>Physical backups: version 5.0.x, 6.0.x, 7.0.x</li><li>Incremental backups: version 5.0.14-12, 6.0.3-2 and higher, 7.0.x</li></ul> | Yes |
| [2.4.0](../release-notes/2.4.0.md) | <ul><li>Logical backups: version 5.0.x and higher</li></ul> | <ul><li>Logical backups: version 5.0.x and higher</li><li>Physical backups: version 5.0.x, 6.0.x, 7.0.x</li><li>Incremental backups: version 5.0.14-12, 6.0.3-2 and higher, 7.0.x</li></ul>| Yes |
| [2.3.0](../release-notes/2.3.0.md) | <ul><li>Logical backups: version 4.4 and higher</li></ul> | <ul><li>Logical backups: version 4.4 and higher</li><li>Physical backups - version 4.4.6-8 and higher, 5.0.x, 6.0.x</li><li>Incremental backups: versions 4.4.18, 5.0.14-12, 6.0.3-2 and higher</li></ul>| Yes |
| [2.2.0](../release-notes/2.2.0.md) | <ul><li>Logical backups: version 4.4 and higher</li></ul> | <ul><li>Logical backups: version 4.4 and higher</li><li>Physical backups - version 4.4.6-8 and higher, 5.0.x, 6.0.x</li><li>Incremental backups: versions 4.4.18, 5.0.14-12, 6.0.3-2 and higher</li></ul>| Yes |
| [2.1.0](../release-notes/2.1.0.md) | <ul><li>Logical backups: version 4.4 and higher</li></ul> | <ul><li>Logical backups: version 4.4 and higher</li><li>Physical backups - version 4.4.6-8, 5.0.x, 6.0.x</li><li>Incremental backups: version 4.2.24-24, 4.4.18, 5.0.14-12, 6.0.3-2 and higher</li></ul>| No. A fresh backup is required|
| [1.7.0](../release-notes/1.7.0.md) | <ul><li>Logical backups: version 4.2 and higher</li></ul> | <ul><li>Logical backups: version 4.2 and higher</li><li>Physical backups (tech preview): version 4.2.15-16, 4.4.6-8, 5.0 and higher</li></ul> | Yes
| [1.6.1](../release-notes/1.6.1.md) | <ul><li>Logical backups: version 3.6 and higher with [MongoDB Replication :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) enabled</li></ul> | <ul><li>Logical backups: version 3.6 and higher</li></ul>|Yes





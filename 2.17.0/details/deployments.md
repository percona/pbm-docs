> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Supported MongoDB deployments

Percona Backup for MongoDB works with sharded clusters and replica sets. It doesn’t work on standalone MongoDB instances. This is because Percona Backup for MongoDB requires [oplog](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/glossary.html#oplog) to guarantee backup consistency. Oplog is available on nodes with replication enabled.

For testing purposes, you can deploy Percona Backup for MongoDB on a single-node replica set. To convert a standalone server into a replica set, specify the `replication.replSetName` option in the configuration file and then initiate the replica set.

Version added: [2.1.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.1.0.html)

[Physical restores](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/physical.html) are available for deployments with arbiter nodes. However, since these nodes don’t have [the `pbm-agent` installed](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/repos.html#what-nodes-to-install-on), you must [manually stop them before the restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore.html#before-you-start).

See also

MongoDB Documentation: [Convert a Standalone to a Replica Set](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)

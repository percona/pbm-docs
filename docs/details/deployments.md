# Supported MongoDB deployments

Percona Backup for MongoDB works with sharded clusters and replica sets. It doesn’t work on standalone MongoDB instances. This is because Percona Backup for MongoDB requires [oplog](../reference/glossary.md#oplog) to guarantee backup consistency. Oplog is available on nodes with replication enabled.

For testing purposes, you can deploy Percona Backup for MongoDB on a single-node replica set. To convert a standalone server into a replica set, specify the `replication.replSetName` option in the configuration file and then initiate the replica set.

!!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

[Physical restores](../features/physical.md) are available for deployments with arbiter nodes. However, since these nodes don't have [the `pbm-agent` installed](../installation.md#what-nodes-to-install-on), you must [manually stop them before the restore](../usage/restore.md#before-you-start).

!!! admonition "See also"

    MongoDB Documentation: [Convert a Standalone to a Replica Set](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)


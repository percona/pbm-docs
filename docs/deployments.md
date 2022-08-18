# Supported MongoDB deployments

Percona Backup for MongoDB doesn’t work on standalone MongoDB instances. This is because Percona Backup for MongoDB requires an [oplog](reference/glossary.md#oplog) to guarantee backup consistency. Oplog is available on nodes with replication enabled.

For testing purposes, you can deploy Percona Backup for MongoDB on a single-node replica set. ( Specify the `replication.replSetName` option in the configuration file of the standalone server.)

!!! admonition "See also"

    MongoDB Documentation: [Convert a Standalone to a Replica Set](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)


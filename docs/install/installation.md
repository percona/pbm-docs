# Install Percona Backup for MongoDB

This section contains information on installing Percona Backup for MongoDB 

## Supported platforms

Percona Backup for MongoDB is available for the most 64-bit Linux distributions. Find the list of supported platforms on the [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mongodb) page.

A Docker image for Percona Backup for MongoDB is also available for ARM64 architectures.

## Supported MongoDB deployments

Percona Backup for MongoDB works with sharded clusters and replica sets. It doesn’t work on standalone MongoDB instances. This is because Percona Backup for MongoDB requires [oplog](reference/glossary.md#oplog) to guarantee backup consistency. Oplog is available on nodes with replication enabled.

For testing purposes, you can deploy Percona Backup for MongoDB on a single-node replica set. To convert a standalone server into a replica set, specify the `replication.replSetName` option in the configuration file.

!!! admonition "See also"

    MongoDB Documentation: [Convert a Standalone to a Replica Set](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)

## Supported MongoDB versions

Percona Backup for MongoDB is compatible with the following MongoDB versions:

* For *logical* backups - [Percona Server for MongoDB](https://www.percona.com/software/mongo-database/percona-server-for-mongodb) and MongoDB Community v4.0 and higher with [MongoDB Replication](https://docs.mongodb.com/manual/replication/) enabled.

* For *physical* backups - [Percona Server for MongoDB](https://www.percona.com/software/mongo-database/percona-server-for-mongodb) starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher with [MongoDB Replication](https://docs.mongodb.com/manual/replication/) enabled and WiredTiger configured as the storage engine.

## Installation tutorials

Choose how you wish to install Percona Backup for MongoDB:

* [from Percona repositories](#installing-from-percona-repositories) using the package manager of your operating system. This is the recommended way

* [build from source code](#building-from-source-code) if you want full control over the installation

* [download packages from Percona website](#download-packages-from-percona-website) and install them using the package manager of your operating system

* [run Percona Backup for MongoDB in a Docker container](https://hub.docker.com/r/percona/percona-backup-mongodb)

After the installation completes, you have the following tools:

| Tool            | Purpose                                                  |
| --------------- | ---------------------------------------------------------|
| `pbm`           | Command-line interface for controlling the backup system |
| `pbm-agent`     | An agent for running backup/restore actions on a database host |
| `pbm-speed-test`| An interface for field-testing compression and backup upload speed|
| `pbm-agent-entrypoint` | An entry point application that allows starting `pbm-agent` and also restarts it in case of any faults| 

## What nodes to install on

### `pbm-agent`

Install `pbm-agent` on all servers that have `mongod` nodes in the
MongoDB cluster (or non-sharded replica set). You don't need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install `pbm` CLI on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.


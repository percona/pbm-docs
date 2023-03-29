# Glossary

## ACID
     
Set of properties that guarantee database transactions are processed reliably. Stands for [`Atomicity`](#atomicity), [`Consistency`](#consistency), [`Isolation`](#isolation), [`Durability`](#durability).

## Amazon S3

Amazon S3 (Simple Storage Service) is an object storage service provided through a web service interface offered by Amazon Web Services.

## Atomicity

Atomicity means that database operations are applied following an "all or nothing" rule. A transaction is either fully applied or not at all.

## Blob
    
A blob stands for Binary Large Object, which includes objects such as images and multimedia files. In other words these are various data files that you store in Microsoft's data storage platform. Blobs are organized in [containers](#container) which are kept in Azure Blob storage under your storage account.

## Bucket

A bucket is a container on the s3 remote storage that stores backups.

## Collection
     
A collection is the way data is organized in MongoDB. It is analogous to a table in relational databases.

## Completion time

Starting with version 2.0.0, the completion time is renamed "restore_to_time"

The completion time is the time to which the sharded cluster / non-shared replica set will be returned to after the restore.  It is reflected in the "complete" section of the ``pbm list`` / ``pbm status`` command outputs.

In `logical` backups, the completion time almost coincides with the backup finish time. To define the completion time, Percona Backup for MongoDB waits for the backup snapshot to finish on all cluster nodes. Then it captures the oplog from the backup start time up to that time. 

In `physical` backups, the completion time is only a few seconds after the backup start time. By holding the ``$backupCursor`` open, Percona Backup for MongoDB guarantees that the checkpoint data won't change during the backup. In such a way Percona Backup for MongoDB can define the completion time ahead.

## Consistency

In the context of backup and restore, consistency means that the data restored will be consistent in a given point in time. Partial or incomplete writes to disk of atomic operations (for example, to table and index data structures separately) won't be served to the client after the restore. The same applies to multi-document transactions that started but didn't complete by the time the backup was finished.

## Container 
   
A container is like a directory in Azure Blob storage that contains a set of [blobs](#blob).

## Durability
   
Once a transaction is committed, it will remain so.

## EBS-snapshot

An EBS (Amazon Elastic Block Storage) snapshot is the point-in-time copy of your data, and can be used to enable disaster recovery, migrate data across regions and accounts, and improve backup compliance.

## GCP
   
GCP (Google Cloud Platform) is the set of services, including storage service, that runs on Google Cloud infrastructure.

## Isolation

The Isolation requirement means that no transaction can interfere with another.

## Jenkins
     
[Jenkins](http://www.jenkins-ci.org) is a continuous integration system that we use to help ensure the continued quality of the software we produce. It helps us achieve the aims of:

* No failed tests in trunk on any platform
* Aid developers in ensuring merge requests build and test on all platforms,
* No known performance regressions (without a damn good explanation).

## MinIO

MinIO is a cloud storage server compatible with [Amazon S3](#amazon-s3), released under Apache License v2.

## Oplog
  
Oplog (operations log) is a fixed-size collection that keeps a rolling record of all operations that modify data in the database. 

## Oplog slice

A compressed bundle of [oplog](#oplog) entries stored in the Oplog Store database in MongoDB. The oplog size captures an approximately 10-minute frame. For a snapshot, the oplog size is defined by the time that the slowest replica set member requires to perform mongodump.    

## OpID

A unique identifier of an operation such as backup, restore, resync. When a pbm-agent starts processing an operation, it acquires a lock and an opID. This prevents processing the same operation twice (for example, if there are network issues in distributed systems). Using opID as a log filter allows viewing logs for an operation in progress.

## `pbm-agent`

A `pbm-agent` is a PBM process running on the mongod node for backup and restore operations. A pbm-agent instance is required for every mongod node (including replica set secondary members and config server replica set nodes).   

## pbm CLI
     
Command-line interface for controlling the backup system. PBM CLI can connect to several clusters so that a user can manage backups on many clusters.

## PBM Control collections
   
PBM Control collections are [collections](#collection) with config, authentication data and backup states. They are stored in the admin db  in the cluster or non-sharded replica set and serve as the communication channel between [`pbm-agent`](#pbm-agent) and [`pbm CLI`](#pbm-cli). `pbm CLI` creates a new `pbmCmd` document for a new operation. `pbm-agents` monitor it and update as they process the operation.

## Percona Backup for MongoDB

Percona Backup for MongoDB (PBM) is a low-impact backup solution for MongoDB non-sharded replica sets and clusters. It supports both [Percona Server for MongoDB](#percona-server-for-mongodb) and MongoDB Community Edition. 

## Percona Server for MongoDB 

Percona Server for MongoDB is a drop-in replacement for MongoDB Community Edition with enterprise-grade features.

## Point-in-Time Recovery
     
Point-in-Time Recovery is restoring the database up to a specific moment in time. The data is restored from the backup snapshot and then events that occurred to the data are replayed from oplog. 

## Replica set
   
A replica set is a group of `mongod` nodes that host the same data set.

## S3 compatible storage 

This is the storage that is built on the [S3](#amazon-s3) API.
 
## Server-side encryption
   
Server-side encryption is the encryption of data by the remote storage server as it receives it. The data is encrypted when it is written to S3 bucket and decrypted when you access the data. 

## Technical preview feature

Technical preview features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA. 


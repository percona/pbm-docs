# System requirements

* At least 1GB RAM is required on every node for `pbm-agents` to operate successfully.
* All `pbm-agents` in the cluster must be able to connect to all config server replica set nodes that could become a new primary. In non-sharded replica set deployments, this means to connect to all the nodes that could become a new primary node. To become a primary, a node must meet the following criteria:

    * have `priority` greater than `0` and must be able to vote (`votes`: 1)
    * is not an arbiter (`arbiterOnly: false`)
    * is not hidden (`hidden: false`)
    * is not delayed 

* All `pbm-agents` in your deployment must be able to connect to the same [remote backup storage](details/storage-configuration.md) using the same credentials.
* All `pbm-agents` and PBM CLI must be of the same version for backups to success. Otherwise we cannot guarantee successful backups and data consistency in them.

Note that networking issues like connection to the remote backup storage can also affect PBM performance. 
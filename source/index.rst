.. _pbm.index:

|pbm| Documentation
********************************************************************************

|pbm| is a distributed, low-impact solution for achieving consistent backups of
|mongodb| sharded clusters and replica sets.

As of version 1.7.0, |pbm| supports both physical and logical backups and restores. :ref:`pitr` is currently supported only for logical backups.

.. important::
 
   Physical backups and restores is the technical preview feature [1]_. Before using them in production, we recommend that you test restoring from physical backups in your environment, and also use an alternative backup method for redundancy.

.. rubric:: Supported MongoDB versions

|PBM| is compatible with the following MongoDB versions:

* For *logical* backups - `Percona Server for MongoDB <https://www.percona.com/software/mongo-database/percona-server-for-mongodb>`_ and MongoDB Community v4.0 and higher with `MongoDB Replication <https://docs.mongodb.com/manual/replication/>`_ enabled.

* For *physical* backups - `Percona Server for MongoDB <https://www.percona.com/software/mongo-database/percona-server-for-mongodb>`_ starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher with `MongoDB Replication <https://docs.mongodb.com/manual/replication/>`_ enabled and WiredTiger configured as the storage engine.

.. important::

   |PBM| doesn't work on standalone |mongodb| instances. This is because |PBM| requires an :term:`oplog <Oplog>` to guarantee backup consistency. Oplog is available on nodes with replication enabled.

   For testing purposes, you can deploy |PBM| on a single-node replica set. ( Specify the ``replication.replSetName`` option in the configuration file of the standalone server.)  

   .. seealso::

      MongoDB Documentation: Convert a Standalone to a Replica Set
         https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/

The |pbm| project is inherited from and replaces `mongodb_consistent_backup`,
which is no longer actively developed or supported.

.. _pbm.feature:

Features
=====================

.. hlist::
   :columns: 2

   - Logical backup and restore
   - Physical (a.k.a. 'hot') backup and restore (with |PSMDB| 4.2.15-16, 4.4.6-8, 5.0.2-1 and higher)
   - Works for both for sharded clusters and classic, non-sharded replica sets.
   - Point-in-time recovery (for logical backups only)
   - Simple command-line management utility
   - Simple, integrated-with-MongoDB authentication
   - Distributed transaction consistency with MongoDB 4.2+
   - Use with any S3-compatible storage
   - Users with classic, locally-mounted remote filesystem backup servers can
     use 'filesystem' instead of 's3' storage type.

Introduction
***********************************************

.. toctree::
   :caption: Introduction
   :maxdepth: 2

   intro
   pbm-components

Getting started
***********************************************

.. toctree::
   :caption: Getting started
   :maxdepth: 2

   installation   
   initial-setup
   

Usage
***********************************************

.. toctree::
   :caption: Usage
   :maxdepth: 2  

   running
   point-in-time-recovery
   selective-backup
   oplog-replay



Details
***********************************************

.. toctree::
   :caption: Details
   :maxdepth: 1

   architecture
   authentication
   backup-types
   config
   storage-configuration

How to 
***********************************************

.. toctree::
   :caption: How to
   :maxdepth: 2
      
   schedule-backup
   Upgrade PBM <upgrading>
   Troubleshoot PBM <troubleshooting>   
   automate-s3-access
   Remove PBM <uninstalling>


FAQ
***********************************************

.. toctree::
   :caption: FAQ
   :maxdepth: 1

   faq

Reference
***********************************************
   
.. toctree::
   :caption: Reference
   :maxdepth: 1

   pbm-commands
   configuration-options
   contributing
   glossary
   copyright
   trademark-policy

Release notes
***********************************************
   
.. toctree::
   :caption: Release notes
   :maxdepth: 1
   
   release-notes

.. include:: .res/replace.txt
.. include:: .res/url.txt

.. [1] Tech Preview Features are not yet ready for enterprise use and are not included in support via |SLA|. They are included in this release so that users can provide feedback prior to the full release of the feature in a future |GA| release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.
         
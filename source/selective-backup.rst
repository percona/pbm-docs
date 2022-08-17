.. _selective-backup:

****************************
Selective Backup and Restore
****************************

.. important::

   Selective backup and restore is the technical preview feature

Starting with version 2.0, you can back up and restore certain namespaces - databases or collections. For example, if your "Payments" collection in the "Staff" database was corrupted, you can restore only this collection from your full backup up to a specific point in time. Or, if your "Invoices" database contains sensitive data and must be backed up frequently, you can configure the backup of only this database. This way you work only with the desired subset of data without disrupting the operations of your whole cluster. 

You also drastically reduce time on backup / restore operations of the whole data set and save on storage consumption.

With the selective backup and restore functionality you have the following options:

1.	Backup a single database or a specific collection and restore all data from it. 
2.	Restore a specific collection from a single database backup
3.	Restore certain databases and / or collections from a full backup
4.	Make a point-in time recovery for the specified databases / collections.


Selective backup 
===========================

To make a selective backup,  run the ``pbm backup`` command and provide the value for the ``--ns`` flag in the format ``<database.collection>``. For example, to back up the "Payments" collection, run the following command:

.. code-block:: bash

   $ pbm backup –ns=staff.payments

To back up the "Invoices" database and all collections that it includes, run the ``pbm backup`` command as follows:

.. code-block:: bash

   $ pbm backup –ns=invoices.*

During the backup process, the |PBM| stores data in the new multi-file format where each collection has a separate file. The oplog is stored for all namespaces regardless whether this is a full or selective backup.

Multi-format is now the default data format for both full and selective backups since it allows selective restore. Note, however, that you can make only full restores from backups made with earlier versions of |PBM|. 

View information about a selective backup 
===============================================

Selective backups are marked as ``selective`` in the ``pbm list`` and ``pbm status`` outputs:

.. code-block:: bash

   $ pbm list

   Backup snapshots:
     2022-08-17T10:03:29Z <physical> [complete: 2022-08-17T10:03:34Z]
     2022-08-17T10:49:03Z <logical, selective> [complete: 2022-08-17T10:49:08Z]


To view a detailed information about a backup, run the following command:

.. code-block:: bash

   $ pbm describe-backup <backup-name>

The output provides the backup name, type, status, size, namespaces and the information about the cluster topology it was taken in:

Output:

.. code-block:: text

   name: "2022-08-17T10:49:03Z"
   type: logical
   last_write_ts:
     t: 1660733348
     i: 2
   namespaces:
   - flight.*
   mongodb_version: 5.0.10-9
   pbm_version: 2.0.0-dev
   status: done
   size: 10234670
   error: ""
   replsets:
   - name: rs1
     status: done
     iscs: false
     last_write_ts:
       t: 1660733348
       i: 2
     error: ""

Selective restore 
====================

To restore a specific database or a collection, run the ``pbm restore`` command in the format:

.. code-block:: bash
 
   $ pbm restore <backup_name> --ns <database.collection>

During the restore, |PBM| retrieves the file for the specified database / collection and restores it.  

Point-in-time recovery 
=============================

To start |PITR| oplog slicing, a full backup snapshot is required. A full backup snapshot also serves as the base for any restore. 

To restore the desired database or a collection to a point in time, run the ``pbm restore`` command as follows:

.. code-block:: bash

   $ pbm restore --base-snapshot <backup_name> --time <timestamp> --ns <db.collection>

When point-in-time recovery it started, |PBM| uses the provided base snapshot, restores the specified namespace and replays oplog on top of it up to the specified time. If no base snapshot is provided, |PBM| uses the most recent one.

Known limitations
======================

1. Only logical backups and restores are supported
#.	Multiple namespaces are not yet supported for selective backups. For restores, you can specify multiple namespaces (e.g. to restore several collections from a database).
#.	Sharded collections are not supported. 
#.	System collections in ``admin``, ``config`` and ``local`` databases cannot be backed up and restored selectively. You must make a full backup and restore to include them.
#.	|PITR| slicing requires a full backup because it serves as the base for |PITR|. Any selective backup will be ignored.


.. include:: .res/replace.txt


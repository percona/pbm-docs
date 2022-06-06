.. _pbm.running:

Running |pbm|
********************************************************************************

This document provides examples of using |pbm.app| commands to operate your backup system. For detailed description of pbm commands, refer to :ref:`pbm-commands`.


.. contents::
   :local:
   :backlinks: top

.. _pbm.running.backup.listing:

Listing backups
--------------------------------------------------------------------------------

To view all completed backups, run the |pbm-list| command.

.. code-block:: bash

   $ pbm list

As of version 1.4.0, the |pbm-list| output shows the completion time. This is the time to which the sharded cluster / non-shared replica set will be returned to after the restore.

.. admonition:: Sample output

   .. code-block:: text

      Backup snapshots:
        2021-01-13T15:50:54Z [complete: 2021-01-13T15:53:40]
        2021-01-13T16:10:20Z [complete: 2021-01-13T16:13:00]
        2021-01-20T17:09:46Z [complete: 2021-01-20T17:10:33]

In `logical` backups, the completion time almost coincides with the backup finish time. To define the completion time, |PBM| waits for the backup snapshot to finish on all cluster nodes. Then it captures the oplog from the backup start time up to that time. 

In `physical` backups, the completion time is only a few seconds after the backup start time. By holding the ``$backupCursor`` open guarantees that the checkpoint data won’t change during the backup, and |PBM| can define the completion time ahead.


.. _pbm.running.backup.starting:

Starting a backup
--------------------------------------------------------------------------------

.. code-block:: bash

   $ pbm backup --type=TYPE

As of version 1.7.0, you can specify what type of a backup you wish to make: physical or logical. 

When `physical` backup is selected, |PBM| copies the contents of the ``dbpath`` directory (data and metadata files, indexes, journal and logs) from every shard and config server replica set to the backup storage.

During `logical` backups, |PBM| copies the actual data to the backup storage. When no ``--type`` flag is passed, |PBM| makes a logical backup.  

For more information about backup types, see :ref:`backup-types`.


By default, |PBM| uses ``s2`` compression method when making a backup. 
You can start a backup with a different compression method by passing the ``--compression`` flag to the |pbm-backup| command. 

For example, to start a backup with gzip compression, use the following command

.. code-block:: bash

   $ pbm backup --compression=gzip 

Supported compression types are: ``gzip``, ``snappy``, ``lz4``, ``pgzip``, ``zstd``.  The ``none`` value means no compression is done during
backup.

As of version 1.7.0, you can configure the compression level for backups. Specify the value for the ``--compression-level`` flag. Note that the higher value you specify, the longer it takes to compress / retrieve the data.

.. rubric:: Backup in sharded clusters

.. important::

   For PBM v1.0 (only): before running |pbm-backup| on a cluster, stop the
   balancer.

In sharded clusters, one of |pbm-agent| processes for every shard and the config server replica set writes backup snapshots  into the remote backup storage directly. For `logical` backups, ``pbm-agents`` also write :term:`oplog slices <Oplog slice>`. To learn more about oplog slicing, see :ref:`pitr`.

The ``mongos`` nodes are not involved in the backup process.

The following diagram illustrates the backup flow.

.. image:: _images/pbm-backup-shard.png

|

.. important::

   If you `reshard <https://www.mongodb.com/docs/manual/core/sharding-reshard-a-collection/>`_ a collection in MongoDB 5.0 and higher versions, make a fresh backup to prevent data inconsistency and restore failure.

.. rubric:: Adjust node priority for backups

In |PBM| prior to version 1.5.0, the ``pbm-agent`` to do a backup is elected randomly among secondary nodes in a replica set. In sharded cluster deployments, the ``pbm-agent`` is elected among the secondary nodes in every shard and the config server replica sets. If no secondary node responds in a defined period, then the ``pbm-agent`` on the primary node is elected to do a backup. 

As of version 1.5.0, you can influence the ``pbm-agent`` election by assigning a priority to ``mongod`` nodes in the |PBM| :ref:`configuration file <pbm.config>`. 

.. code-block:: yaml

   backup:
     priority:
       "localhost:28019": 2.5
       "localhost:27018": 2.5
       "localhost:27020": 2.0
       "localhost:27017": 0.1

The format of the `priority` array is ``<hostname:port>``:``<priority>``.

To define priority in a sharded cluster, you can either list all nodes or specify priority for one node in each shard and config server replica set. The hostname and port uniquely identifies a node so that |PBM| recognizes where it belongs to and grants the priority accordingly.

Note that if you listed only specific nodes, the remaining nodes will be automatically assigned priority 1.0. For example, you assigned priority 2.5 to only one secondary node in every shard and config server replica set of the sharded cluster. 

.. code-block:: yaml

   backup:
     priority:
       "localhost:27027": 2.5  # config server replica set
       "localhost:27018": 2.5  # shard 1
       "localhost:28018": 2.5  # shard 2

The remaining secondaries and the primary nodes in the cluster receive priority 1.0.

The ``mongod`` node with the highest priority makes the backup. If this node is unavailable, next priority node is selected. If there are several nodes with the same priority, one of them is randomly elected to make the backup. 

If you haven't listed any nodes for the ``priority`` option in the config, the nodes have the default priority for making backups as follows: 

- hidden nodes - priority 2.0
- secondary nodes - priority 1.0
- primary node - priority 0.5

This ability to adjust node priority helps you manage your backup strategy by selecting specific nodes or nodes from preferred data centers. In geographically distributed infrastructures, you can reduce network latency by making backups from nodes in geographically closest locations.

.. important::

   As soon as you adjust node priorities in the configuration file, it is assumed that you take manual control over them. The default rule to prefer secondary nodes over primary stops working.

Checking an in-progress backup
--------------------------------------------------------------------------------

.. important::

   As of version 1.4.0, the information about running backups is not available in the |pbm-list| output. Use the :command:`pbm status` command instead to check for running backups. See :ref:`pbm-status` for more information.

For |PBM| version 1.3.4 and earlier, run the |pbm-list| command and you will see the running backup listed with a
'In progress' label. When that is absent, the backup is complete.

As of version 1.7.0, the ``pbm list`` output includes the type of backup.

.. code-block:: bash

   $ pbm list

     Backup snapshots:
       2021-12-13T13:05:14Z <physical> [complete: 2021-12-13T13:05:17]


.. _pbm.running.backup.restoring:

Restoring a backup
--------------------------------------------------------------------------------

.. warning::

   Backups made with |PBM| prior to v1.5.0 are incompatible for restore with |PBM| v1.5.0 and later. This is because processing of system collections ``Users`` and ``Roles`` has changed: in v1.5.0, ``Users`` and ``Roles`` are copied to temporary collection during backup and must be present in the backup during restore. In earlier versions of |PBM|, ``Users`` and ``Roles`` are copied to a temporary collection during restore. Therefore, restoring from these backups with |PBM| v1.5.0 isn't possible. 

   The recommended approach is to make a fresh backup after :ref:`upgrading Percona Backup for MongoDB <pbm.upgrade>` to version 1.5.0.


To restore a backup that you have made using |pbm-backup|, use the
|pbm-restore| command supplying the time stamp of the backup that you intend to
restore. Percona Backup for MongoDB identifies the type of the backup (physical or logical) and restores the database up to the backup completion time (available in pbm list output as of version 1.4.0).

.. important::

   Consider these important notes on restore operation:

   1. |pbm| is designed to be a full-database restore tool. As of version <=1.x, it performs a full all-databases, all collections restore and does not offer an option to restore only a subset of collections in the backup, as MongoDB's ``mongodump`` tool does. But to avoid surprising ``mongodump`` users, as of versions 1.x, |pbm| replicates mongodump's behavior to only drop collections in the backup. It does not drop collections that are created new after the time of the backup and before the restore. Run a ``db.dropDatabase()`` manually in all non-system databases (these are all databases except "local", "config" and "admin") before running |pbm-restore| if you want to guarantee that the post-restore database only includes collections that are in the backup.
   2. Whilst the restore is running, prevent clients from accessing the database. The data will naturally be incomplete whilst the restore is in progress, and writes the clients make cause the final restored data to differ from the backed-up data.
   3. If you enabled :term:`Point-in-Time Recovery`, disable it before running |pbm-restore|. This is because |PITR| incremental backups and restore are incompatible operations and cannot be run together.

.. code-block:: bash

   $ pbm restore 2019-06-09T07:03:50Z

.. rubric:: Adjust memory consumption

.. versionadded:: 1.3.2

   The |pbm| config includes the restore options to adjust the memory consumption by the |pbm-agent| in environments with tight memory bounds. This allows preventing out of memory errors during the restore operation.

.. code-block:: yaml

   restore:
     batchSize: 500
     numInsertionWorkers: 10

The default values were adjusted to fit the setups with the memory allocation of 1GB and less for the agent.

.. note::

  The lower the values, the less memory is allocated for the restore. However, the performance decreases too.

Restoring a backup in sharded clusters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. important::

   As preconditions for restoring a backup in a sharded cluster, complete the following steps:

   1. Stop the balancer.
   2. Shut down all ``mongos`` nodes to stop clients from accessing the database while restore is in progress. This ensures that the final restored data doesn’t differ from the backed-up data.
   3. Disable point-in-time recovery if it is enabled. To learn more about point-in-time recovery, see :ref:`pitr`.

Note that you can restore a sharded backup only into a sharded environment. It can be your existing cluster or a new one. To learn how to restore a backup into a new environment, see :ref:`pbm.restore-new-env`.

During the restore, the |pbm-agent| processes write data to primary nodes in the cluster. The following diagram shows the restore flow.

.. image:: _images/pbm-restore-shard.png

|

After a cluster's restore is complete, restart all ``mongos`` nodes to reload the sharding metadata.

.. admonition:: Physical restore known limitations

   Tracking restore progress via ``pbm status`` is currently not available during physical restores. To check the restore status, the options are:
   
   - Check the stderr logs of the leader |pbm-agent|. The leader ID is printed once the restore has started.
   - Check the status in the metadata file created on the remote storage for the restore. This file is in the root of the storage path and has the format ``.pbm.restore/<restore_timestamp>.json``
     
   After the restore is complete, do the following:

   - Restart all ``mongod`` nodes
   - Restart all ``pbm-agents`` 
   - Run the following command to resync the backup list with the storage:
     
     .. code-block::

        $ pbm config --force-resync

.. _pbm.restore-new-env:

Restoring a backup into a new environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To restore a backup from one environment to another, consider the following key points about the destination environment:

* Replica set names (both the config servers and the shards) in your new destination cluster and in the cluster that was backed up must be exactly the same.

* |PBM| configuration in the new environment must point to the same remote storage that is defined for the original environment, including the authentication credentials if it is an object store. Once you run |pbm-list| and see the backups made from the original environment, then you can run the |pbm-restore| command.

  Of course, make sure not to run |pbm-backup| from the new environment whilst the |PBM| config is pointing to the remote storage location of the original environment.

Restoring into a cluster / replica set with a different name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Starting with version 1.8.0, you can restore **logical backups** into a new environment that has the same or more number of shards and these shards have different replica set names. 

To restore data to the environment with different replica set names, configure the name mapping between the source and target environments. You can either set the ``PBM_REPLSET_REMAPPING`` environment variable for ``pbm`` CLI or use the ``--replset-remapping`` flag for PBM commands. The mapping format is ``<rsTarget>=<rsSource>``.

.. important::

   Configure replica set name mapping for all shards in your cluster. Otherwise, |PBM| attempts to restore the unspecified shard to the target shard with the same name. If there is no shard with such name or it is already mapped to another source shard, the restore fails.

Configure the replica set name mapping:

* Using the environment variable for ``pbm`` CLI in your shell:

  .. code-block:: bash

     $ export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"

* Using the command line:
  
  .. code-block:: bash

     $ pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"

The ``--replset-remapping`` flag is available for the following commands: ``pbm restore``, ``pbm list``, ``pbm status``, ``pbm oplog-replay``.

.. note::

   Don't forget to make a fresh backup on the new environment after the restore is complete. 

This ability to restore data to clusters with different replica set names and the number of shards extends the set of environments compatible for the restore.  



.. _pbm.cancel.backup:

Canceling a backup
--------------------------------------------------------------------------------

You can cancel a running backup if, for example, you want to do
another maintenance of a server and don't want to wait for the large backup to finish first.

To cancel the backup, use the :command:`pbm cancel-backup` command.

.. code-block:: bash

  $ pbm cancel-backup
  Backup cancellation has started

After the command execution, the backup is marked as canceled in the :command:`pbm status` output:

.. code-block:: bash

  $ pbm status
  ...
  2020-04-30T18:05:26Z	Canceled at 2020-04-30T18:05:37Z

.. _pbm.backup.delete:

Deleting backups
--------------------------------------------------------------------------------

Use the :command:`pbm delete-backup` command to delete a specified backup or all backups
older than the specified time.

The command deletes the backup regardless of the remote storage used:
either S3-compatible or a filesystem-type remote storage.

.. note::

  You can only delete a backup that is not running (has the "done" or the "error" state). 


  As of version 1.4.0, |pbm-list| shows only successfully completed backups. To check for backups with other states, run :command:`pbm status`. 

To delete a backup, specify the ``<backup_name>`` as an argument.

.. code-block:: bash

  $ pbm delete-backup 2020-12-20T13:45:59Z

By default, the :command:`pbm delete-backup` command asks for your confirmation
to proceed with the deletion. To bypass it, add the ``-f`` or
``--force`` flag.

.. code-block:: bash

  $ pbm delete-backup --force 2020-04-20T13:45:59Z

To delete backups that were created before the specified time, pass the ``--older-than`` flag to the :command:`pbm delete-backup`
command. Specify the timestamp as an argument
for :command:`pbm delete-backup` in the following format:

* ``%Y-%M-%DT%H:%M:%S`` (for example, 2020-04-20T13:13:20) or
* ``%Y-%M-%D`` (2020-04-20).

.. code-block:: bash

 $ #View backups
 $ pbm list
 Backup snapshots:
   2020-04-20T20:55:42Z   
   2020-04-20T23:47:34Z
   2020-04-20T23:53:20Z
   2020-04-21T02:16:33Z
 $ #Delete backups created before the specified timestamp
 $ pbm delete-backup -f --older-than 2020-04-21
 Backup snapshots:
   2020-04-21T02:16:33Z

.. _pbm.logs:

Viewing backup logs
--------------------------------------------------------------------------------

As of version 1.4.0, you can see the logs from all ``pbm-agents`` in your MongoDB environment using ``pbm CLI``. This reduces time for finding required information when troubleshooting issues.

.. note::

   The log information about restores from physical backups not available in pbm logs.

To view |pbm-agent| logs, run the :command:`pbm logs` command and pass one or several flags to narrow down the search.

The following flags are available:

-	``-t``, ``--tail`` - Show the last N rows of the log
-	``-e``, ``--event`` - Filter logs by all backups or a specific backup
-	``-n``, ``--node`` - Filter logs by a specific node  or a replica set
-	``-s``, ``--severity`` - Filter logs by severity level. The following values are supported (from low to high):

   - D - Debug
   - I - Info
   - W - Warning
   - E - Error
   - F - Fatal
   
- ``-o``, ``--output`` - Show log information as text (default) or in JSON format.
- ``-i``, ``--opid`` - Filter logs by the operation ID

.. rubric:: Examples

The following are some examples of filtering logs:

**Show logs for all backups**

.. code-block:: bash

   $ pbm logs --event=backup

**Show the last 100 lines of the log about a specific backup 2020-10-15T17:42:54Z**

.. code-block:: bash

   $ pbm logs --tail=100 --event=backup/2020-10-15T17:42:54Z

**Include only errors from the specific replica set**

.. code-block:: bash

   $ pbm logs -n rs1 -s E

The output includes log messages of the specified severity type and all higher levels. Thus, when ERROR is specified, both ERROR and FATAL messages are shown in the output.

.. rubric:: Implementation details

``pbm-agents`` write log information into the ``pbmLog`` collection in the :term:`PBM Control collections`. Every |pbm-agent| also writes log information to stderr so that you can retrieve it when there is no healthy mongod node in your cluster or replica set. For how to view an individual |pbm-agent| log, see :ref:`pbm-agent.log`.

Note that log information from ``pbmLog`` collection is shown in the UTC timezone and from the stderr - in the server's time zone.

.. include:: .res/replace.txt

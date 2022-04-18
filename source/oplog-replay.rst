.. _oplog-replay:

*******************
|PITR| oplog replay
*******************

Starting with version 1.7.0, you can replay the :term:`oplog <Oplog>` for a specific period on top of any backup: logical, physical, storage level snapshot (like EBS-snapshot). In this way you can manually restore sharded clusters and non sharded replica sets to a specific point in time from a backup made by any tool and not only by |PBM|.

.. warning::

   Use the oplog replay functionality with caution, only when you are sure about the starting time to replay oplog from. The oplog replay does not guarantee data consistency when restoring from any backup, however, it is less error-prone for backups made with |PBM|.

Oplog replay for physical backups
=================================

To replay the oplog on top of physical backups made with |PBM|, do the following:

1.	Stop |PITR|, if enabled, to release the lock. 
2.	Run ``pbm status`` or ``pbm list`` commands to find oplog chunks available for replay. 
3.	Run the ``pbm oplog-replay`` command and specify the ``--start`` and ``--end`` flags with the timestamps. 

    .. code-block:: bash

       $ pbm oplog-replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"

4. After the oplog replay, make a fresh backup and enable the |PITR| oplog slicing.
   
Oplog replay for storage level snapshots
=========================================

When making a backup, |PBM| stops the |PITR|. This is done to maintain data consistency after the restore. 

Storage-level snapshots are saved with |PITR| enabled. Thus, after the database restore from such a backup, |PITR| is automatically enabled and starts oplog slicing. These new oplog slices might conflict with the existing oplogs saved during the backup. To replay the oplog in such a case, do the following after the restore:

1. Disable |PITR|
#. Delete the oplog slices that might have been created
#. Resync the data from the storage
#. Run the ``pbm oplog-replay`` command and specify the ``--start`` and ``--end`` flags with the timestamps.
#. After the oplog replay, make a fresh backup and enable the |PITR| oplog slicing.


.. admonition:: Known limitations

   The oplog replay fails if you rename the entire database or a collection.


.. include:: .res/replace.txt

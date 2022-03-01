.. _oplog-replay:

*******************
|PITR| oplog replay
*******************

Starting with version 1.7.0, you can replay the :term:`oplog <Oplog>` for a specific period on top of any backup: logical, physical, storage level snapshot (like EBS-snapshot). In this way you can restore sharded clusters and non sharded replica sets to a specific point in time from a backup made by any tool and not only by |PBM|.

To replay the oplog, do the following:

1.	Stop |PITR|, if enabled, to release the lock. 
2.	Run ``pbm status`` or ``pbm list`` commands to find oplog chunks available for replay. 
3.	Run the ``pbm oplog replay`` command and specify the ``--start`` and ``--end`` flags with the timestamps. 

    .. code-block:: bash

       $ pbm oplog replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"

4.	After the oplog replay, make a fresh backup and enable the |PITR| oplog slicing

.. admonition:: Known limitations

   The oplog replay fails if you rename the entire database or a collection.


.. include:: .res/replace.txt
.. _pbm.intro:

How |pbm| works
********************************************************************************

Even in a highly-available architecture, such as with |mongodb| replication, backups are still required even though losing one server is not fatal. Whether for a complete or partial data disaster, you can use PBM (Percona Backup for MongoDB) to go back in time to the best available backup snapshot.

|PBM| is a command line interface. It provides the set of commands to make backup and restore operations in your database. 

The following example illustrates how to use |PBM|.

With |PBM| up and running in your environment, make a backup:

.. code-block:: bash

   $ pbm backup

To also save all events that occurred to the data between the backups, enable saving oplog slices:

.. code-block:: bash

   $ pbm config --set pitr.enabled=true

.. tip::

   You can :ref:`schedule the frequency of backups <schedule>` via a cron task 

Now, imagine that your web application’s update was released on February 7th 03:00 UTC. By 15:23 UTC, someone realizes that this update has a bug that is wiping the historical data of any user who logged in. To remediate this negative impact on data, it’s time to roll back up to the time of the application’s update - up to February 7th, 03:00 UTC

.. code-block:: bash

   $ pbm list

.. admonition:: Output

   .. code-block:: text

      Backup snapshots:
        2021-02-03T08:08:15Z [complete: 2021-02-03T08:08:35Z]
        2021-02-05T13:55:55Z [complete: 2021-02-05T13:56:15Z]
        2021-02-07T13:57:58Z [complete: 2021-02-07T13:58:17Z]
        2021-02-09T14:06:06Z [complete: 2021-02-09T14:06:26Z]
        2021-02-11T14:22:41Z [complete: 2021-02-11T14:23:01Z]

      PITR <on>:
       2021-02-03T08:08:36Z-2021-02-09T12:20:23Z
       2021-02-09T14:06:27Z-2021-02-11T14:22:40Z

The output lists the valid time ranges for the restore. The desired time (February 7th, 03:00 UTC) falls within the ``2021-02-03T08:08:36Z-2021-02-09T12:20:23Z`` range, so let’s restore the database up to that time. 

Since the restore and saving oplog slices are exclusive operations and cannot run together, let’s stop oplog slicing first:

.. code-block:: bash

   $ pbm config --set pitr.enabled=false

Now, restore the database:

.. code-block:: bash

   $ pbm restore --time 2021-02-07T02:59:59

To be on the safe side, it is a good practice to make a fresh backup after the restore is complete.

.. code-block:: bash

   $ pbm backup

This backup refreshes the timeline and serves as the base for saving oplog slices. To re-enable this process, run:

.. code-block:: bash 

   $ pbm config --set pitr.enabled=true

Find the full set of commands available in |PBM| in :ref:`pbm-commands`.

.. include:: .res/replace.txt

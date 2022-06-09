.. _pbm.troubleshooting:

==============================================
Troubleshooting |pbm|
==============================================

|pbm| provides troubleshooting tools to operate data backups.

.. contents::
   :local:
   :depth: 1

pbm-speed-test
==============================================

|pbm-speed-test| allows field-testing compression and backup upload speed. You
can use it:

* to check performance before starting a backup;
* to find out what slows down the running backup.

By default, |pbm-speed-test| operates with fake semi random data documents. To
run |pbm-speed-test| on a real collection, provide a valid :ref:`MongoDB connection URI string <pbm.auth.mdb_conn_string>` for the ``--mongodb-uri`` flag.

Run :command:`pbm-speed-test` for the full set of available commands.

Compression test
----------------------------------------------

.. code-block:: bash

   $ pbm-speed-test compression --compression=s2 --size-gb 10
   Test started ....
   10.00GB sent in 8s.
   Avg upload rate = 1217.13MB/s.


:command:`pbm-speed-test compression` uses the compression library from the config
file and sends a fake semi random data document (1 GB by default) to the
black hole storage. (Use the ``pbm config`` command to change the compression library). 

To test compression on a real collection, pass the
``--sample-collection`` flag with the <my_db.my_collection> value.

Run |pbm-speed-test-compression-help| for the full set of supported flags:

.. code-block:: bash

  $ pbm-speed-test compression --help
  usage: pbm-speed-test compression

  Run compression test

  Flags:
        --help                     Show context-sensitive help (also try
                                   --help-long and --help-man).
        --mongodb-uri=MONGODB-URI  MongoDB connection string
    -c, --sample-collection=SAMPLE-COLLECTION  
                                   Set collection as the data source
    -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
        --compression=s2           Compression type
                                   <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
        --compression-level=COMPRESSION-LEVEL 
                                   Compression level (specific to the compression type)                               
                                   <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>



Upload speed test
----------------------------------------------------

.. code-block:: bash

   $ pbm-speed-test storage --compression=s2
   Test started 
   1.00GB sent in 1s.
   Avg upload rate = 1744.43MB/s.

|pbm-speed-test-storage| sends the semi random data (1 GB by default) to the
remote storage defined in the config file. Pass the ``--size-gb`` flag to change the
data size. 

To run the test with the real collection's data instead of the semi random data,
pass the ``--sample-collection`` flag with the <my_db.my_collection> value.

Run |pbm-speed-test-storage-help| for the full set of available flags:

.. code-block:: bash

   $ pbm-speed-test storage --help
   usage: pbm-speed-test storage

   Run storage test

   Flags:
         --help                     Show context-sensitive help (also try --help-long and --help-man).
         --mongodb-uri=MONGODB-URI  MongoDB connection string
     -c, --sample-collection=SAMPLE-COLLECTION  
                                    Set collection as the data source
     -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
         --compression=s2           Compression type <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
         --compression-level=COMPRESSION-LEVEL 
                                   Compression level (specific to the compression type)                               


Backup progress tracking
============================================================================

If you have a large backup you can track backup progress in |pbm-agent| logs. A line is appended every 
minute showing bytes copied vs. total size for the current collection.

.. include:: .res/code-block/bash/pbm-agent-backup-progress-log.txt

.. _pbm-status:

|PBM| status
============================================================================

As of version 1.4.0, you can check the status of |pbm| running in your MongoDB environment using the :command:`pbm status` command. 

.. code-block:: bash

   $ pbm status

The output provides the information about:

- Your MongoDB deployment and ``pbm-agents`` running in it: to what ``mongod`` node each agent is connected, the |pbm| version it runs and the agent's state
- The currently running backups / restores, if any
- Backups stored in the remote backup storage: backup name, :ref:`completion time <Completion time>`, size and status (complete, canceled, failed)
- :term:`Point-in-Time Recovery` status (enabled or disabled).
- Valid time ranges for point-in-time recovery and the data size

This simplifies troubleshooting since the whole information is provided in one place.

.. admonition:: Sample output

   .. code-block:: text

      $ pbm status

      Cluster:
      ========
      config:
        - config/localhost:27027: pbm-agent v1.3.2 OK
        - config/localhost:27028: pbm-agent v1.3.2 OK
        - config/localhost:27029: pbm-agent v1.3.2 OK
      rs1:
        - rs1/localhost:27018: pbm-agent v1.3.2 OK
        - rs1/localhost:27019: pbm-agent v1.3.2 OK
        - rs1/localhost:27020: pbm-agent v1.3.2 OK
      rs2:
        - rs2/localhost:28018: pbm-agent v1.3.2 OK
        - rs2/localhost:28019: pbm-agent v1.3.2 OK
        - rs2/localhost:28020: pbm-agent v1.3.2 OK

      PITR incremental backup:
      ========================
      Status [OFF]

      Currently running:
      ==================
      (none)

      Backups:
      ========
      S3 us-east-1 https://storage.googleapis.com/backup-test
         Snapshots:
           2020-12-16T10:36:52Z 491.98KB [complete: 2020-12-16T10:37:13Z]
           2020-12-15T12:59:47Z 284.06KB [complete: 2020-12-15T13:00:08Z]
           2020-12-15T11:40:46Z 0.00B [canceled: 2020-12-15T11:41:07Z]
           2020-12-11T16:23:55Z 284.82KB [complete: 2020-12-11T16:24:16Z]
           2020-12-11T16:22:35Z 284.04KB [complete: 2020-12-11T16:22:56Z]
           2020-12-11T16:21:15Z 283.36KB [complete: 2020-12-11T16:21:36Z]
           2020-12-11T16:19:54Z 281.73KB [complete: 2020-12-11T16:20:15Z]
           2020-12-11T16:19:00Z 281.73KB [complete: 2020-12-11T16:19:21Z]
           2020-12-11T15:30:38Z 287.07KB [complete: 2020-12-11T15:30:59Z]
      PITR chunks:
           2020-12-16T10:37:13 - 2020-12-16T10:43:26 44.17KB

.. _backup-log:

``pbm-agent`` logs
============================================================================

To troubleshoot issues with specific events or node(s), use the :ref:`logs` command.  It provides logs of all ``pbm-agent`` processes in your environment. The command is available as of version 1.4.0.

:command:`pbm logs` has the set of filters to refine logs for specific events like ``backup``, ``restore``, ``pitr`` or for a specific node, and to manage log verbosity level. For example, to view logs about a specific backup with the Debug verbosity level, run the ``pbm logs`` command as follows:

.. code-block:: bash

   $ pbm logs --severity=D --event=backup/2020-10-15T17:42:54Z 

To learn more about available filters and usage examples, refer to :ref:`pbm.logs`.

.. include:: .res/replace.txt

.. _initial-setup:

Initial setup   
****************************************************************

The following diagram shows the overview of the installation and setup steps: 

.. uml:: _images/setup.puml
   :align: center

Refer to :ref:`install` for installation instructions. Remember to install |pbm-agent| on every server with a ``mongod`` node that is not an arbiter node.

The setup steps are the following:

1. :ref:`Configure authentication in MongoDB <authenticate>`
#. :ref:`Insert the config information for remote backup storage <backup-config>`
#. :ref:`Start pbm-agent process <start-pbm-agent>`
 	

.. _authenticate:

Configure authentication in MongoDB
================================================================================

|pbm| has no authentication and authorization subsystem of its own - it uses
the one of MongoDB. This means that to authenticate |pbm|, you need to create a corresponding ``pbm`` user in the ``admin`` database and set a valid MongoDB connection URI string for both |pbm-agent| and ``pbm`` CLI.

.. _pbm.auth.create_pbm_user:

Create the ``pbm`` user
---------------------------------------

First create the role that allows any action on any resource. 

.. code-block:: javascript

   db.getSiblingDB("admin").createRole({ "role": "pbmAnyAction",
         "privileges": [
            { "resource": { "anyResource": true },
              "actions": [ "anyAction" ]
            }
         ],
         "roles": []
      });

Then create the user and assign the role you created to it. 

.. code-block:: javascript

   db.getSiblingDB("admin").createUser({user: "pbmuser",
          "pwd": "secretpwd",
          "roles" : [
             { "db" : "admin", "role" : "readWrite", "collection": "" },
             { "db" : "admin", "role" : "backup" },
             { "db" : "admin", "role" : "clusterMonitor" },
             { "db" : "admin", "role" : "restore" },
             { "db" : "admin", "role" : "pbmAnyAction" }
          ]
       });

You can specify username and password values and other options of the ``createUser`` command as you require so long as the roles shown above are granted.

Create the ``pbm`` user on every replica set. In a sharded cluster, this means every shard replica set and the config server replica set.

.. note::

   In a cluster run `db.getSiblingDB("config").shards.find({}, {"host": true,
   "_id": false})` to list all the host+port lists for the shard
   replica sets. The replica set name at the *front* of these "host" strings will
   have to be placed as a "/?replicaSet=xxxx" argument in the parameters part
   of the connection URI (see below).

Set the MongoDB connection URI for ``pbm-agent``
------------------------------------------------------------------

A |pbm-agent| process connects to its localhost ``mongod`` node with a standalone type of connection. To set the MongoDB URI connection string means to configure a service init script (:file:`pbm-agent.service` systemd unit file) that runs a |pbm-agent|.

The :file:`pbm-agent.service` systemd unit file includes the environment file. You set the MongoDB URI connection string for the  ``PBM_MONGODB_URI`` variable within the environment file.

The environment file for Debian and Ubuntu is :file:`/etc/default/pbm-agent`. For Redhat and CentOS, it is :file:`/etc/sysconfig/pbm-agent`. 

Edit the environment file and specify MongoDB connection URI string for the ``pbm`` user to the local ``mongod`` node. For example, if ``mongod`` node listens on port 27018, the MongoDB connection URI string will be the following:

.. code-block:: bash

   PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27018/?authSource=admin"

.. note:: 

   If the password includes special characters like ``#``, ``@``, ``/`` and so on, you must convert these characters using the `percent-encoding mechanism <https://datatracker.ietf.org/doc/html/rfc3986#section-2.1>`_ when passing them to |PBM|. For example, the password ``secret#pwd`` should be passed as follows in ``PBM_MONGODB_URI``:

   .. code-block:: text

      PBM_MONGODB_URI="mongodb://pbmuser:secret%23pwd@localhost:27018/?authSource=admin"

Configure the service init script for every |pbm-agent|. 

.. hint:: **How to find the environment file**

   The path to the environment file is specified in the :file:`pbm-agent.service` systemd unit file. 

   In Ubuntu and Debian, the :file:`pbm-agent.service` systemd unit file is at the path :file:`/lib/systemd/system/pbm-agent.service`. 

   In Red Hat and CentOS, the path to this
   file is :file:`/usr/lib/systemd/system/pbm-agent.service`.
       
   .. admonition:: Example of pbm-agent.service systemd unit file  

      .. include:: .res/code-block/bash/systemd-unit-file.txt    

.. seealso::

   More information about standard MongoDB connection strings
      :ref:`pbm.auth`

.. _set-mongodbURI-pbm.CLI: 

Set the MongoDB connection URI for ``pbm CLI``
------------------------------------------------------------------

Provide the MongoDB URI connection string for |pbm.app| in your shell. This allows you to call |pbm.app| commands without the ``--mongodb-uri`` flag.

Use the following command:

.. code-block:: bash
 
   export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27018/?authSource=admin&replSetName=xxxx"

For more information what connection string to specify, refer to :ref:`pbm.auth.pbm.app_conn_string` section.

External authentication support in |PBM|
------------------------------------------------------------------

In addition to SCRAM, |PBM| supports other `authentication method <https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html>`_ that you use in MongoDB or |PSMDB|. 

For external authentication, you :ref:`create the pbm user <pbm.auth.create_pbm_user>` in the format used by the authentication system and set the MongoDB connection URI string to include both the authentication method and authentication source.

For example, for `Kerberos authentication <https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html#kerberos-authentication>`_, create the ``pbm`` user in the ``$external`` database in the format ``<username@KERBEROS_REALM>`` (e.g. pbm@PERCONATEST.COM).

Specify the following string for MongoDB connection URI:

.. code-block:: bash

   PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"

Note that you must first obtain the ticket for the ``pbm`` user with the ``kinit`` command before you start the |pbm-agent|:

.. code-block:: bash

   $ sudo -u {USER} kinit pbm  

Note that the ``{USER}`` is the user that you will run the ``pbm-agent`` process.


For `authentication and authorization via Native LDAP <https://docs.percona.com/percona-server-for-mongodb/latest/authorization.html#authentication-and-authorization-with-direct-binding-to-ldap>`_, you only create roles for LDAP groups in MongoDB as the users are stored and managed on the LDAP server. However, you still define the ``$external`` database as your authentication source: 

.. code-block:: bash

   PBM_MONGODB_URI="mongodb://<user>:<password>@<hostname>:27018/?authMechanism=PLAIN&authSource=%24external&replSetName=xxxx"



.. _backup-config:

Configure remote backup storage
==================================

The easiest way to provide remote backup storage configuration is to specify it in a YAML config file and upload this file to |PBM| using ``pbm CLI``.

The storage configuration itself is out of scope of the present document. We assume that you have configured one of the :ref:`supported remote backup storages <storage.config>`.

.. important::

      * When using a remote backup storage (S3 or Microsoft Azure), grant the  ``List/Get/Put/Delete`` permissions to the storage for the user identified by the access credentials. Please refer to the documentation of your selected storage for the permissions configuration.
      * When using a filesystem storage, verify that the user running |PBM| is the owner of the backup directory. The recommended user is `mongod`, so it should be the owner of the backup directory.

        .. code-block:: bash
        
           $ sudo chown mongod:mongod <backup_directory>

1. Create a config file (e.g. :file:`pbm_config.yaml`).
2. Specify the storage information within. 
   
   The following is the sample configuration for Amazon AWS:
   
   .. include:: .res/code-block/yaml/example-amazon-s3-storage.yaml
   
   This is the sample configuration for Microsoft Azure Blob storage:

   .. include:: .res/code-block/yaml/example-azure-storage.yaml

   This is the sample configuration for filesystem storage:

   .. include:: .res/code-block/yaml/example-local-file-system-store.yaml

   See more examples in :ref:`pbm.config.example_yaml`.

3. Insert the config file
   
   .. code-block:: bash

      $ pbm config --file pbm_config.yaml 

   For a sharded cluster, run this command whilst connecting to config server replica set. Otherwise connect to the non-sharded replica set as normal. 

To learn more about |PBM| configuration, see :ref:`pbm.config`.
 
.. _start-pbm-agent:

Start the |pbm-agent| process
==================================

Start |pbm-agent| on every server with the ``mongod`` node installed. It is best to use the packaged service scripts to run |pbm-agent|. 

.. code-block:: bash

   $ sudo systemctl start pbm-agent
   $ sudo systemctl status pbm-agent

.. admonition:: Example

   Imagine you put configsvr nodes (listen port 27019) collocated on the same
   servers as the first shard's ``mongod`` nodes (listen port 27018, replica set name "sh1rs"). In this server there should be two |pbm-agent| processes, one connected to the shard (e.g. "mongodb://username:password@localhost:27018/") and one to the configsvr node (e.g. "mongodb://username:password@localhost:27019/").

For reference, the following is an example of starting |pbm-agent| manually. The
output is redirected to a file and the process is backgrounded. 

.. attention::
   
   Start the ``pbm-agent`` as the ``mongod`` user. The ``pbm-agent`` requires write access to the MongoDB data directory to make physical restores.

.. code-block:: bash

   $ su mongod nohup pbm-agent --mongodb-uri "mongodb://username:password@localhost:27018/" > /data/mdb_node_xyz/pbm-agent.$(hostname -s).27018.log 2>&1 &

Replace ``username`` and ``password`` with those of your ``pbm`` user. ``/data/mdb_node_xyz/`` is the path where |pbm-agent| log files will be written. Make sure you have created this directory and granted write permissions to it for the ``mongod`` user. 

Alternatively,
you can run ``pbm-agent`` on a shell terminal temporarily if you want to observe and/or
debug the startup from the log messages.

.. _pbm-agent.log:

How to see the pbm-agent log
--------------------------------------------------------------------------------

With the packaged systemd service, the log output to stdout is captured by
systemd's default redirection to systemd-journald. You can view it with the
command below. See :command:`man journalctl` for useful options such as ``--lines``, ``--follow``, etc.

.. code-block:: bash

   ~$ journalctl -u pbm-agent.service
   -- Logs begin at Tue 2019-10-22 09:31:34 JST. --
   Jan 22 15:59:14 akira-x1 systemd[1]: Started pbm-agent.
   Jan 22 15:59:14 akira-x1 pbm-agent[3579]: pbm agent is listening for the commands
   ...
   ...

If you started |pbm-agent| manually, see the file you redirected stdout and stderr
to.

When a message *"pbm agent is listening for the commands"* is printed to the
|pbm-agent| log file, |pbm-agent| confirms that it has connected to its ``mongod`` node successfully.


.. include:: .res/replace.txt

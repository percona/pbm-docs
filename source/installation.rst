.. _pbm.installation:
.. _install:

Installing |pbm|
********************************************************************************

.. contents::
   :local:

|percona| provides and supports |pbm| installation packages for Debian, Ubuntu, Red Hat Enterprise Linux and CentOS Linux distributions. Find detailed information about supported Linux distributions on the `Percona Software and Platform Lifecycle <https://www.percona.com/services/policies/percona-software-platform-lifecycle#mongodb>`_ page.

You can install |pbm| in one of the following ways:

- :ref:`from Percona repositories (recommended) <repo>`
- :ref:`build from source code <source-code>` if you want full control over the installation
- :ref:`download packages from Percona website <tarball>` and install them using the package manager of your operating system

Alternatively, you can run Percona Backup for MongoDB `in a Docker container <https://hub.docker.com/r/percona/percona-backup-mongodb>`_

Regardless of the installation method you choose, the following tools are at
your disposal after the installation completes:

===============  ===============================================================
Tool             Purpose
===============  ===============================================================
pbm              Command-line interface for controlling the backup system
pbm-agent        An agent for running backup/restore actions on a database host
pbm-speed-test   An interface for field-testing compression and backup upload 
                 speed
===============  ===============================================================

|

Install |pbm-agent| on every server that has ``mongod`` nodes in the
MongoDB cluster (or non-sharded replica set). You don't need to install |pbm-agent| on arbiter nodes since they don't have the data set.

You can install |pbm.app| CLI 
on any or all servers or desktop computers you wish to use it from, so long as
those computers aren't network-blocked from accessing the MongoDB cluster.

.. _repo:

Installing from Percona repositories
================================================================================

This is the recommended installation method. Percona provides the ``percona-release`` configuration tool that simplifies operating repositories and enables to install and update both |pbm| packages and required dependencies smoothly.

Install |percona-release| tool using the package manager of your operating system. Follow the instructions in `Percona Software repositories documentation <https://www.percona.com/doc/percona-repo-config/installing.html>`_ to install |percona-release|.

Enable the repository. As of version 1.3.0, |pbm| packages are stored in the *pbm* repository.

.. code-block:: bash

   $ sudo percona-release enable pbm release

Install |pbm| on Debian and Ubuntu 
--------------------------------------------------

Reload the local package database:

.. code-block:: bash

   $ sudo apt update

Install |pbm|:

.. code-block:: bash

   $ sudo apt install percona-backup-mongodb

Install |pbm| on Red Hat Enterprise Linux and CentOS 
----------------------------------------------------------

Use the following command to install |pbm|:

.. code-block:: bash

   $ sudo yum install percona-backup-mongodb

.. _source-code:

Building from source code
================================================================================

Building the project requires:

- Go 1.15 or above
- make
- git
- ``krb5-devel`` for Red Hat Enterprise Linux / CentOS or ``libkrb5-dev`` for Debian / Ubuntu. This package is required for Kerberos authentication in Percona Server for MongoDB.

.. seealso::

   Installing and setting up Go tools
      https://golang.org/doc/install

To build the project (from the project dir):

.. code-block:: bash

   $ git clone https://github.com/percona/percona-backup-mongodb
   $ cd percona-backup-mongodb
   $ make build

After :program:`make` completes, you can find |pbm.app| and |pbm-agent| binaries
in the :dir:`./bin` directory:

.. code-block:: bash

   $ cd bin
   $ ./pbm version

By running :program:`pbm version`, you can verify if |pbm| has been built correctly and is ready for use.

.. admonition:: Output

   .. code-block:: bash

      Version:   [pbm version number]
      Platform:  linux/amd64
      GitCommit: [commit hash]
      GitBranch: main
      BuildTime: [time when this version was produced in UTC format]
      GoVersion: [Go version number]

.. tip::

   Instead of specifying the path to pbm binaries, you can add it to the ``PATH`` environment variable:

   .. code-block:: bash
   
      export PATH=/percona-backup-mongodb/bin:$PATH

Post-install steps
-------------------------

1. Create the environment file:

   * The path for Debian and Ubuntu is :file:`/etc/default/pbm-agent`. 
   * The path for RHEL and CentOS is :file:`/etc/sysconfig/pbm-agent`. 

#. Create the :file:`pbm-agent.service` systemd unit file. 

   In Ubuntu and Debian, the :file:`pbm-agent.service` systemd unit file is at the path :file:`/lib/systemd/system/pbm-agent.service`. 

   In RHEL and CentOS, the path to this
   file is :file:`/usr/lib/systemd/system/pbm-agent.service`.

#. In the :file:`pbm-agent.service` file, specify the following:
   
   .. code-block:: init

      [Unit]
      Description=pbm-agent
      After=time-sync.target network.target

      [Service]
      EnvironmentFile=-/etc/default/pbm-agent
      Type=simple
      User=mongod
      Group=mongod
      PermissionsStartOnly=true
      ExecStart=/usr/bin/pbm-agent

      [Install]
      WantedBy=multi-user.target
       
   .. note::

      Make sure that the ``ExecStart`` directory includes the |PBM| binaries. Otherwise, copy them from the ``./bin`` directory of you installation path.

#. Make ``systemd`` aware of the new service:
   
   .. code-block:: bash

      $ sudo systemctl daemon-reload

.. _tarball:

Download packages from Percona website
========================================

You can `download installation packages <https://www.percona.com/downloads/percona-backup-mongodb/>`_ specific for your operating system from Percona website and install them using ``dpkg`` (Debian and Ubuntu) or ``rpm`` (Red Hat Enterprise Linux and CentOS). However, you must make sure that all dependencies are satisfied.

Alternatively, you can download and install |pbm| from binary tarballs.

Install from binary tarball
---------------------------

Find the link to the binary tarballs under the *Generic Linux* menu item on `Percona website <https://www.percona.com/downloads/percona-backup-mongodb/>`_.

1. Fetch the binary tarball. Replace the <version> with the required version.
   
   .. code-block:: bash

      $ wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-<version>/binary/tarball/percona-backup-mongodb-<version>-x86_64.tar.gz

2. Extract the tarball

   .. code-block:: bash

      $ tar -xf percona-backup-mongodb-<version>-x86_64.tar.gz

3. Export the location of the binaries to the ``PATH`` variable. For example, if you've extracted the tarball to your ``home`` directory, the command would be the following:
   
   .. code-block:: bash

      $ export PATH=~/percona-backup-mongodb-<version>/:$PATH


After |pbm| is successfully installed on your system, you have |pbm.app|
and |pbm-agent| programs available. See :ref:`initial-setup` for guidelines how to set up |PBM|.  


.. include:: .res/replace.txt

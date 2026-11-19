> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Build from source code

## What nodes to install on

### `pbm-agent`

Install [`pbm-agent`](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/pbm-agent.html) on all servers that have `mongod` nodes in the MongoDB cluster (or non-sharded replica set). You don’t need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install [`pbm` CLI](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/cli.html) on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.

## Before you start

Check the [system requirements](https://docs.percona.com/percona-backup-mongodb/2.17.0/system-requirements.html) and [supported MongoDB versions](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/versions.html).

## Prerequisites

To build Percona Backup for MongoDB from source, you need the following:

- Go 1.19 or above. [Install and set up Go tools](https://golang.org/doc/install)
- make
- git
- `krb5-devel` for Red Hat Enterprise Linux / CentOS or `libkrb5-dev` for Debian / Ubuntu. This package is required for Kerberos authentication in Percona Server for MongoDB.

## Procedure

Here’s how to build Percona Backup for MongoDB:

1. Clone the repository

   ```
   git clone https://github.com/percona/percona-backup-mongodb
   ```

1. Go to the project directory and build it

   ```
   cd percona-backup-mongodb
   make build
   ```

After **make** completes, you can find `pbm` and `pbm-agent` binaries in the `./bin` directory.

1. Check that Percona Backup for MongoDB has been built correctly and is ready for use.

   ```
   cd bin
   ./pbm version
   ```

   Output

   ```
   Version:   [pbm version number]
   Platform:  linux/amd64
   GitCommit: [commit hash]
   GitBranch: main
   BuildTime: [time when this version was produced in UTC format]
   GoVersion: [Go version number]
   ```

   Tip

   Instead of specifying the path to pbm binaries, you can add it to the PATH environment variable:

   ```
   export PATH=/percona-backup-mongodb/bin:$PATH
   ```

## Post-install steps

**On Debian and Ubuntu**

After the installation, do the following:

1. Create the environment file:

   ```
   touch /etc/default/pbm-agent
   ```

1. Create the `pbm-agent.service` systemd unit file.

   ```
   sudo vim /lib/systemd/system/pbm-agent.service
   ```

1. In the `pbm-agent.service` file, specify the following:

   ```
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
   ```

   Note

   Make sure that the `ExecStart` directory includes the Percona Backup for MongoDB binaries. Otherwise, copy them from the `./bin` directory of you installation path.

1. Make `systemd` aware of the new service:

   ```
   sudo systemctl daemon-reload
   ```

**On Red Hat Enterprise Linux and derivatives**

After the installation, do the following:

1. Create the environment file:

   ```
   touch /etc/sysconfig/pbm-agent
   ```

1. Create the `pbm-agent.service` systemd unit file.

   ```
   sudo vim /usr/lib/systemd/system/pbm-agent.service
   ```

1. In the `pbm-agent.service` file, specify the following:

   ```
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
   ```

   Note

   Make sure that the `ExecStart` directory includes the Percona Backup for MongoDB binaries. Otherwise, copy them from the `./bin` directory of you installation path.

1. Make `systemd` aware of the new service:

   ```
   sudo systemctl daemon-reload
   ```

## Next steps

[Initial setup](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Install from Percona repositories

To install the software from Percona repositories means to subscribe to them. Percona provides the [`percona-release`](https://www.percona.com/doc/percona-repo-config/index.html) repository management tool. It automatically enables the required repository so that you can install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

## What nodes to install on

### `pbm-agent`

Install [`pbm-agent`](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/pbm-agent.html) on all servers that have `mongod` nodes in the MongoDB cluster (or non-sharded replica set). You don’t need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install [`pbm` CLI](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/cli.html) on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.

## Before you start

Check the [system requirements](https://docs.percona.com/percona-backup-mongodb/2.17.0/system-requirements.html) and [supported MongoDB versions](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/versions.html).

## Procedure

*Warning:* Run the following commands as root or via the `sudo` command.

### Configure Percona repository

Percona provides the [`percona-release`](https://docs.percona.com/percona-software-repositories/index.html) configuration tool that simplifies operating repositories and enables to install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

1. [Install `percona-release`](https://www.percona.com/doc/percona-repo-config/installing.html) .

1. Enable the repository

   ```
   sudo percona-release enable pbm release
   ```

### Install Percona Backup for MongoDB packages

**On Debian and Ubuntu**

1. Reload the local package database:

   ```
   sudo apt update
   ```

1. Install Percona Backup for MongoDB:

   ```
   sudo apt install percona-backup-mongodb
   ```

**On RHEL and derivatives**

Run the following command to install the packages:

```
sudo yum install percona-backup-mongodb
```

**On Amazon Linux 2023**

Run the following command to install the packages:

```
sudo yum install percona-backup-mongodb
```

After the installation completes, you have the following tools:

| Tool                   | Purpose                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------ |
| `pbm`                  | Command-line interface for controlling the backup system                                               |
| `pbm-agent`            | An agent for running backup/restore actions on a database host                                         |
| `pbm-speed-test`       | An interface for field-testing compression and backup upload speed                                     |
| `pbm-agent-entrypoint` | An entry point application that allows starting `pbm-agent` and also restarts it in case of any faults |

## Next steps

[Initial setup](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html)

> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Download Percona Backup for MongoDB from Percona website

You can download Percona Backup for MongoDB from [Percona website](https://www.percona.com/downloads/percona-backup-mongodb/) and install it:

- [From binary tarballs](#install-from-binary-tarballs).
- Manually, from the installation packages using `dpkg` (Debian and Ubuntu) or `rpm` (Red Hat Enterprise Linux and CentOS). However, you must make sure that all dependencies are satisfied.

## What nodes to install on

### `pbm-agent`

Install [`pbm-agent`](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/pbm-agent.html) on all servers that have `mongod` nodes in the MongoDB cluster (or non-sharded replica set). You don’t need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install [`pbm` CLI](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/cli.html) on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.

## Before you start

Check the [system requirements](https://docs.percona.com/percona-backup-mongodb/2.17.0/system-requirements.html) and [supported MongoDB versions](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/versions.html).

## Install from binary tarballs

Find the link to the binary tarballs under the **Generic Linux** menu item on [Percona website](https://www.percona.com/downloads/percona-backup-mongodb/) .

1. Fetch the binary tarball. Replace the version in the URL with your required version.

   ```
   wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-2.17.0/binary/tarball/percona-backup-mongodb-2.17.0-x86_64.tar.gz
   ```

1. Extract the tarball

   ```
   tar -xf percona-backup-mongodb-2.17.0-x86_64.tar.gz
   ```

1. Export the location of the binaries to the `PATH` variable

   For example, if you’ve extracted the tarball to your `home` directory, the command would be the following:

   ```
   export PATH=~/percona-backup-mongodb-2.17.0/:$PATH
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

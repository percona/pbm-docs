> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Upgrade Percona Backup for MongoDB

The recommended and most convenient way to upgrade PBM is from Percona repositories.

## Important notes

1. Backward compatibility between data backup and restore is supported for upgrades within one major version only (for example, from 2.1.x to 2.2.y). When you upgrade Percona Backup for MongoDB across major versions (for example, from 2.0.x to 2.2.y), we recommend to make a backup right after the upgrade.
1. Upgrade Percona Backup for MongoDB on all nodes where it is installed.

## Prerequisites

1. [Install `percona-release` tool](https://www.percona.com/doc/percona-repo-config/installing.html) or [update it](https://www.percona.com/doc/percona-repo-config/updating.html) to the latest version.

1. Enable the repository

   ```
   sudo percona-release enable pbm release
   ```

*Note:* For `apt`-based systems, run `sudo apt update` to update the local cache.

## Upgrade to the latest version

**On Debian and Ubuntu Linux**

Run all commands as root or via `sudo`.

1. Stop `pbm-agent`

   ```
   sudo systemctl stop pbm-agent
   ```

1. Install new packages

   ```
   sudo apt install percona-backup-mongodb
   ```

1. Reload the `systemd` process

   ```
   sudo systemctl daemon-reload
   ```

1. Update permissions

   For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

1. Start `pbm-agent`

   ```
   sudo systemctl start pbm-agent
   ```

**On Red Hat Enterprise Linux and derivatives**

Run all commands as root or via `sudo`.

1. Stop `pbm-agent`

   ```
   sudo systemctl stop pbm-agent
   ```

1. Install new packages

   ```
   sudo yum install percona-backup-mongodb
   ```

1. Reload the `systemd` process

Starting from v1.7.0, reload the `systemd` process to update the unit file with the following command:

```
sudo systemctl daemon-reload
```

1. Update permissions

   For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

1. Start `pbm-agent`

   ```
   sudo systemctl start pbm-agent
   ```

## Upgrade to a specific version

**On Debian and Ubuntu Linux**

Run all commands as root or via `sudo`.

1. List available versions

   ```
   sudo apt-cache madison percona-backup-mongodb
   ```

   Sample output

   ```
   percona-backup-mongodb | 2.8.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
   percona-backup-mongodb | 2.7.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
   percona-backup-mongodb | 2.6.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
   percona-backup-mongodb | 2.5.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
   ```

1. Stop `pbm-agent`

   ```
   sudo systemctl stop pbm-agent
   ```

1. Install packages

   Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.0, run the following command:

   ```
   sudo apt install percona-backup-mongodb=1.7.0-1.stretch
   ```

1. Update permissions

For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

1. Start `pbm-agent`

   ```
   sudo systemctl start pbm-agent
   ```

**On Red Hat Enterprise Linux and derivatives**

Run all commands as root or via `sudo`.

1. List available versions

   ```
   sudo yum list percona-backup-mongodb --showduplicates
   ```

   Sample output

   ```
   Available Packages
   percona-backup-mongodb.x86_64    1.8-1.el7            pbm-release-x86_64
   percona-backup-mongodb.x86_64    1.8.0-1.el7          pbm-release-x86_64
   percona-backup-mongodb.x86_64    1.7.0-1.el7          pbm-release-x86_64
   percona-backup-mongodb.x86_64    1.6.1-1.el7          pbm-release-x86_64
   percona-backup-mongodb.x86_64    1.6.0-1.el7          pbm-release-x86_64
   percona-backup-mongodb.x86_64    1.5.0-1.el7          pbm-release-x86_64
   ```

1. Stop `pbm-agent`

   ```
   sudo systemctl stop pbm-agent
   ```

1. Install packages

   Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.1, run the following command:

   ```
   sudo yum install percona-backup-mongodb-1.7.1-1.el7
   ```

1. Update permissions

   For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

1. Start `pbm-agent`

   ```
   sudo systemctl start pbm-agent
   ```

*Note:* If MongoDB runs under a *different user than `mongod`* (the default configuration for Percona Server for MongoDB), use the same user to run the `pbm-agent`. For filesystem-based storage, grant the read / write permissions to the backup directory for this user.

# Upgrade Percona Backup for MongoDB

The recommended and most convenient way to upgrade PBM is from Percona repositories.

## Important notes

1. Backward compatibility between data backup and restore is supported for upgrades within one major version only (for example, from 2.1.x to 2.2.y). When you upgrade Percona Backup for MongoDB over several major versions (for example, from 2.0.x to 2.2.y), we recommend to make a backup right after the upgrade.

2. Upgrade Percona Backup for MongoDB on all nodes where it is installed.

## Prerequisites 

1. [Install `percona-release` tool :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/installing.html) or [update it :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/updating.html) to the latest version.

2. Enable the repository

    ```{.bash data-prompt="$"}
    $ sudo percona-release enable pbm release
    ```

<i info>:material-information: Note:</i> For `apt`-based systems, run `sudo apt update` to update the local cache.

## Upgrade to the latest version

=== ":material-debian: On Debian and Ubuntu Linux"

    Run all commands as root or via `sudo`.
    {.power-number}

    1. Stop `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl stop pbm-agent
        ```

    2. Install new packages

        ```{.bash data-prompt="$"}
        $ sudo apt install percona-backup-mongodb
        ```  

    3. Reload the `systemd` process

        ```{.bash data-prompt="$"}
        $ sudo systemctl daemon-reload
        ```

    4. Update permissions

        For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    5. Start `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl start pbm-agent
        ```

=== ":material-redhat: On Red Hat Enterprise Linux and derivatives"

    Run all commands as root or via `sudo`.
    {.power-number}

    1. Stop `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl stop pbm-agent
        ```

    2. Install new packages

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-backup-mongodb
        ```

    3. Reload the `systemd` process

       Starting from v1.7.0, reload the `systemd` process to update the unit file with the following command:

       ```{.bash data-prompt="$"}
       $ sudo systemctl daemon-reload
       ```

    4. Update permissions

        For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

    5. Start `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl start pbm-agent
        ``` 

## Upgrade to a specific version

=== ":material-debian: On Debian and Ubuntu Linux"

    Run all commands as root or via `sudo`.
    {.power-number}

    1. List available versions
 
        ```{.bash data-prompt="$"}
        $ sudo apt-cache madison percona-backup-mongodb
        ```

        ??? example "Sample output"

            ```{.text .no-copy}
            percona-backup-mongodb | 2.8.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.7.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.6.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.5.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            ```

    2. Stop `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl stop pbm-agent
        ```

    3. Install packages

        Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.0, run the following command:

        ```{.bash data-prompt="$"}
        $ sudo apt install percona-backup-mongodb=1.7.0-1.stretch
        ```
 
    4. Update permissions

       For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    5. Start `pbm-agent`
 
        ```{.bash data-prompt="$"}
        $ sudo systemctl start pbm-agent
        ``` 

=== ":material-redhat: On Red Hat Enterprise Linux and derivatives"
  
    Run all commands as root or via `sudo`.
    {.power-number}

    1. List available versions

        ```{.bash data-prompt="$"}
        $ sudo yum list percona-backup-mongodb --showduplicates
        ```

        ??? example "Sample output"

            ```{.text .no-copy}
            Available Packages
            percona-backup-mongodb.x86_64    1.8-1.el7            pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.8.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.7.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.6.1-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.6.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.5.0-1.el7          pbm-release-x86_64
            ```

    2. Stop `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl stop pbm-agent
        ```

    3. Install packages

        Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.1, run the following command:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-backup-mongodb-1.7.1-1.el7
        ```
    
    4. Update permissions

        For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.

    5. Start `pbm-agent`

        ```{.bash data-prompt="$"}
        $ sudo systemctl start pbm-agent
        ``` 

<i info>:material-information: Note:</i> If MongoDB runs under a *different user than `mongod`* (the default configuration for Percona Server for MongoDB), use the same user to run the `pbm-agent`. For filesystem-based storage, grant the read / write permissions to the backup directory for this user.

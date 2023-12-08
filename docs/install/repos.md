# Install from Percona repositories

To install the software from Percona repositories means to subscribe to them. Percona provides the [`percona-release`](https://www.percona.com/doc/percona-repo-config/index.html) repository management tool. It automatically enables the required repository so that you can install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

--8<-- "pbm-install-nodes.md"

## Procedure

<i warning>:material-alert: Warning:</i> Run the following commands as root or via the `sudo` command.
{.power-number}

1. [Install `percona-release`](https://www.percona.com/doc/percona-repo-config/installing.html). If you have installed it before, [update](https://www.percona.com/doc/percona-repo-config/updating.html) it to the latest version.

2. Enable the repository

    ```{.bash data-prompt="$"}
    $ sudo percona-release enable pbm release
    ```

3. Install Percona Backup for MongoDB packages

    === "On Debian and Ubuntu"    

        1. Reload the local package database:    

            ```{.bash data-prompt="$"}
            $ sudo apt update
            ```    

        2. Install Percona Backup for MongoDB:    

            ```{.bash data-prompt="$"}
            $ sudo apt install percona-backup-mongodb
            ```    

    === "On Red Hat Enterprise Linux and derivatives" 

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-backup-mongodb
        ```

--8<-- "install-result.md"

## Next steps

[Initial setup :material-arrow-right:](initial-setup.md){.md-button}
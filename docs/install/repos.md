# Install from Percona repositories

To install the software from Percona repositories means to subscribe to them. Percona provides the [`percona-release` :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/index.html) repository management tool. It automatically enables the required repository so that you can install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

--8<-- "pbm-install-nodes.md"

## Before you start

Check the [system requirements](../system-requirements.md).

## Procedure

<i warning>:material-alert: Warning:</i> Run the following commands as root or via the `sudo` command.
{.power-number}

### Configure Percona repository

=== "x86_64"

    Percona provides the [`percona-release` :material-arrow-top-right: ](https://docs.percona.com/percona-software-repositories/index.html) configuration tool that simplifies operating repositories and enables to install and update both Percona Backup for MongoDB packages and required dependencies smoothly. 

    1. [Install `percona-release` :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/installing.html).    

    2. Enable the repository    

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable pbm release
        ```

=== "ARM64"

    === ":material-debian: On Debian and Ubuntu"

        Create the `/etc/apt/sources.list.d/percona-pbm-release.list ` configuration file with the following contents:

        ```ini title='/etc/apt/sources.list.d/percona-pbm-release.list'
        deb http://repo.percona.com/pbm/apt OPERATING_SYSTEM main
        ```
    
    === ":material-redhat: On Red Hat Enterprise Linux and derivatives"

        Create the `/etc/yum.repos.d/percona-pbm-release.repo` configuration file with the following contents:

        ```ini title='/etc/yum.repos.d/percona-pbm-release.repo'
        [pbm-release-aarch64]
        name = Percona Backup MongoDB release/aarch64 YUM repository
        baseurl = http://repo.percona.com/pbm/yum/release/$releasever/RPMS/aarch64
        enabled = 1
        gpgcheck = 1
        gpgkey = file:///etc/pki/rpm-gpg/PERCONA-PACKAGING-KEY
        ```

### Install Percona Backup for MongoDB packages

=== ":material-debian: On Debian and Ubuntu"    

    1. Reload the local package database:    

        ```{.bash data-prompt="$"}
        $ sudo apt update
        ```    

    2. Install Percona Backup for MongoDB:    

        ```{.bash data-prompt="$"}
        $ sudo apt install percona-backup-mongodb
        ```    

=== ":material-redhat: On Red Hat Enterprise Linux and derivatives" 

    ```{.bash data-prompt="$"}
    $ sudo yum install percona-backup-mongodb
    ```

--8<-- "install-result.md"

## Next steps

[Initial setup :material-arrow-right:](initial-setup.md){.md-button}
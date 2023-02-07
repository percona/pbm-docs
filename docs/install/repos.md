# Install from Percona repositories

Use the package manager of your operating system to install Percona Backup for MongoDB:

* `apt` - for Debian and Ubuntu Linux
* `yum` - for Red Hat Enterprise Linux and compatible Linux derivatives

To install the software from Percona repositories means to subscribe to them. Percona provides the [`percona-release`](https://www.percona.com/doc/percona-repo-config/index.html) repository management tool. It automatically enables the required repository for you and enables you to install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

!!! important

    Run the following commands as root or via the `sudo` command

#### 1. Install `percona-release`

[Install `percona-release` tool](https://www.percona.com/doc/percona-repo-config/installing.html). If you have installed it before, [update](https://www.percona.com/doc/percona-repo-config/updating.html) it to the latest version.


#### 2. Enable the repository

Starting with of version 1.3.0, Percona Backup for MongoDB packages are stored in the *pbm* repository.

```sh
sudo percona-release enable pbm release
```

#### 3. Install Percona Backup for MongoDB

=== "On Debian and Ubuntu"

    1. Reload the local package database:

        ```sh
        sudo apt update
        ```

    2. Install Percona Backup for MongoDB:

        ```sh
        sudo apt install percona-backup-mongodb
        ```

=== "On Red Hat Enterprise Linux and derivatives"
    
    Install Percona Backup for MongoDB:

    ```sh
    sudo yum install percona-backup-mongodb
    ```

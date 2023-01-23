# Installing Percona Backup for MongoDB

Install Percona Backup for MongoDB  in one of the following ways:

* [from Percona repositories](#installing-from-percona-repositories) using the package manager of your operating system. This is the recommended way

* [build from source code](#building-from-source-code) if you want full control over the installation

* [download packages from Percona website](#download-packages-from-percona-website) and install them using the package manager of your operating system
* [run Percona Backup for MongoDB in a Docker container](https://hub.docker.com/r/percona/percona-backup-mongodb)

Find the list of supported Linux distributions on the [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mongodb) page.

After the installation completes, you have the following tools:

| Tool            | Purpose                                                  |
| --------------- | ---------------------------------------------------------|
| `pbm`           | Command-line interface for controlling the backup system |
| `pbm-agent`     | An agent for running backup/restore actions on a database host |
| `pbm-speed-test`| An interface for field-testing compression and backup upload speed|
| `pbm-agent-entrypoint` | An entry point application that allows starting `pbm-agent` and also restarts it in case of any faults| 

## What nodes to install on

### `pbm-agent`

Install `pbm-agent` on all servers that have `mongod` nodes in the
MongoDB cluster (or non-sharded replica set). You don't need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install `pbm` CLI on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.

## Install from Percona repositories

Use the package manager of your operating system to install Percona Backup for MongoDB:

* `apt` - for Debian and Ubuntu Linux
* `yum` - for Red Hat Enterprise Linux and compatible Linux derivatives

Percona provides the [`percona-release`](https://www.percona.com/doc/percona-repo-config/index.html) configuration tool that simplifies operating repositories and enables to install and update both Percona Backup for MongoDB packages and required dependencies smoothly.

!!! important

    Run the following commands as root or via the `sudo` command

#### 1. Install `percona-release`

[Install `percona-release` tool](https://www.percona.com/doc/percona-repo-config/installing.html). If you have installed it before, [update](https://www.percona.com/doc/percona-repo-config/updating.html) it to the latest version.


#### 2. Enable the repository

As of version 1.3.0, Percona Backup for MongoDB packages are stored in the *pbm* repository.

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

## Building from source code

### Prerequisites 

Building the project requires:

* Go 1.19 or above
* make
* git
* `krb5-devel` for Red Hat Enterprise Linux / CentOS or `libkrb5-dev` for Debian / Ubuntu. This package is required for Kerberos authentication in Percona Server for MongoDB.

!!! admonition "See also"

    [Install and set up Go tools](https://golang.org/doc/install)

    

### Procedure

#### 1. Clone the repository

```sh
git clone https://github.com/percona/percona-backup-mongodb
```

#### 2. Go to the project directory and build it

```sh
cd percona-backup-mongodb
make build
```

After **make** completes, you can find `pbm` and `pbm-agent` binaries
in the `./bin` directory:

```sh
cd bin
./pbm version
```

To verify if Percona Backup for MongoDB has been built correctly and is ready for use, run

```sh
pbm version
```

Output

```text
Version:   [pbm version number]
Platform:  linux/amd64
GitCommit: [commit hash]
GitBranch: main
BuildTime: [time when this version was produced in UTC format]
GoVersion: [Go version number]
```

!!! tip

    Instead of specifying the path to pbm binaries, you can add it to the PATH environment variable:

    ```sh
    export PATH=/percona-backup-mongodb/bin:$PATH
    ```

### Post-install steps

=== "On Debian and Ubuntu"

     1. Create the environment file:

         ```sh
         touch /etc/default/pbm-agent
         ```

     2. Create the `pbm-agent.service` systemd unit file.

         ```sh
         sudo vim /lib/systemd/system/pbm-agent.service
         ```

     3. In the `pbm-agent.service` file, specify the following:

         ```init
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
         
        !!! note

            Make sure that the `ExecStart` directory includes the Percona Backup for MongoDB binaries. Otherwise, copy them from the `./bin` directory of you installation path.

     4. Make `systemd` aware of the new service:

         ```sh
         sudo systemctl daemon-reload
         ```

=== "On Red Hat Enterprise Linux and derivatives"

    1. Create the environment file:
   
        ```sh
        touch /etc/sysconfig/pbm-agent
        ```

    2. Create the `pbm-agent.service` systemd unit file.

        ```sh
        sudo vim /usr/lib/systemd/system/pbm-agent.service
        ```

    3. In the `pbm-agent.service` file, specify the following:

         ```init
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
         
        !!! note

            Make sure that the `ExecStart` directory includes the Percona Backup for MongoDB binaries. Otherwise, copy them from the `./bin` directory of you installation path.

     4. Make `systemd` aware of the new service:

         ```sh
         sudo systemctl daemon-reload
         ```

## Download packages from Percona website

You can download Percona Backup for MongoDB from [Percona website](https://www.percona.com/downloads/percona-backup-mongodb/) and install it:

* [From binary tarballs](#install-from-binary-tarball).
* Manually, from the installation packages using `dpkg` (Debian and Ubuntu) or `rpm` (Red Hat Enterprise Linux and CentOS). However, you must make sure that all dependencies are satisfied.

* Download and install Percona Backup for MongoDB [from binary tarballs](#install-from-binary-tarball).

## Install from binary tarball

Find the link to the binary tarballs under the **Generic Linux** menu item on [Percona website](https://www.percona.com/downloads/percona-backup-mongodb/).


#### 1. Fetch the binary tarball

Replace the `<version>` with the required version.

   ```sh
   wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-<version>/binary/tarball/percona-backup-mongodb-<version>-x86_64.tar.gz
   ```


#### 2. Extract the tarball

  ```sh
  tar -xf percona-backup-mongodb-<version>-x86_64.tar.gz
  ```

#### 3. Export the location of the binaries to the `PATH` variable

For example, if you’ve extracted the tarball to your `home` directory, the command would be the following:

  ```sh
  export PATH=~/percona-backup-mongodb-<version>/:$PATH
  ```

After Percona Backup for MongoDB is successfully installed on your system, you have `pbm` and `pbm-agent` programs available. See [Initial setup](initial-setup.md) for guidelines how to set up Percona Backup for MongoDB.

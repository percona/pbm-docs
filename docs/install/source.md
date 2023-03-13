# Build from source code

## Prerequisites 

Building the project requires:

* Go 1.19 or above
* make
* git
* `krb5-devel` for Red Hat Enterprise Linux / CentOS or `libkrb5-dev` for Debian / Ubuntu. This package is required for Kerberos authentication in Percona Server for MongoDB.

!!! admonition "See also"

    [Install and set up Go tools](https://golang.org/doc/install)

    

## Procedure

### 1. Clone the repository

```sh
git clone https://github.com/percona/percona-backup-mongodb
```

### 2. Go to the project directory and build it

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

## Post-install steps

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

## Next steps

[Set up PBM](initial-setup.md)
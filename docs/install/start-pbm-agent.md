# Start the `pbm-agent` process

Start `pbm-agent` on every server with the `mongod` node installed. 

=== ":material-console: Using `systemd`"

    We recommend to use the packaged service scripts to run `pbm-agent`.
    
    ```{.bash data-prompt="$"}
    $ sudo systemctl start pbm-agent
    $ sudo systemctl status pbm-agent
    ```

=== ":fontawesome-solid-user-gear: Manually"

    You can start a `pbm-agent` manually. You must start a `pbm-agent` as the `mongod` user because the `pbm-agent` requires write access to the MongoDB data directory to make physical restores. 

    The following command starts the `pbm-agent`, redirects the output to a file and puts the process in the background:
    
    ```{.bash data-prompt="$"}
    $ su mongod nohup pbm-agent --mongodb-uri "mongodb://username:password@localhost:27018/" > /data/mdb_node_xyz/pbm-agent.$(hostname -s).27018.log 2>&1 &
    ```
    
    Replace `username` and `password` with those of your `pbm` user. `/data/mdb_node_xyz/` is the path where `pbm-agent` log files will be written. Make sure you have created this directory and granted write permissions to it for the `mongod` user.
    
    Alternatively, you can run `pbm-agent` on a shell terminal temporarily if you want to observe and/or debug the startup from the log messages.

## Multiple agents on the same host

Let's say you run a config server (listen port `27019`) on the same host as another `mongod` process (listen port `27018`). 

In this case there should be two `pbm-agent` processes:

* one process is connected to the `mongod` process (e.g. `“mongodb://username:password@localhost:27018/”`) 
* another one - to the configsvr node (e.g. `“mongodb://username:password@localhost:27019/”`).

The steps below show how to achieve this:

1. Set the MongoDB connection string URI for each agent:

    ```{.bash data-prompt="$"}
    $ tee /etc/sysconfig/pbm-agent1 <<EOF
    PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27018/?authSource=admin"
    EOF
    ```

    ```{.bash data-prompt="$"}
    $ tee /etc/sysconfig/pbm-agent2 <<EOF
    PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27019/?authSource=admin"
    EOF
    ```

2. Prepare service files for each agent:

    ```{.bash data-prompt="$"}
    $ tee /usr/lib/systemd/system/pbm-agent1.service <<EOF
    [Unit]
    Description=pbm-agent for mongod1
    After=time-sync.target network.target    

    [Service]
    EnvironmentFile=-/etc/sysconfig/pbm-agent1
    Type=simple
    User=mongod
    Group=mongod
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent    

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    ```{.bash data-prompt="$"}
    $ tee /usr/lib/systemd/system/pbm-agent2.service <<EOF
    [Unit]
    Description=pbm-agent for mongod2
    After=time-sync.target network.target    

    [Service]
    EnvironmentFile=-/etc/sysconfig/pbm-agent2
    Type=simple
    User=mongod
    Group=mongod
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent    

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

3. Reload system units and start `pbm-agent` processes: 

    ```{.bash data-prompt="$"}
    $ sudo systemctl daemon-reload
    $ sudo systemctl start pbm-agent1
    $ sudo systemctl start pbm-agent2
    ```

## How to see the `pbm-agent` log

With the packaged `systemd` service, the log output to `stdout` is captured by
systemd’s default redirection to `systemd-journald`. You can view it with the
command below. See `man journalctl` for useful options such as `--lines`, `--follow`, etc.

```{.bash data-prompt="$"}
$ journalctl -u pbm-agent.service
-- Logs begin at Tue {{year}}-01-22 09:31:34 JST. --
Jan 22 15:59:14 : Started pbm-agent.
Jan 22 15:59:14 pbm-agent[3579]: pbm agent is listening for the commands
...
...
```

If you started `pbm-agent` manually, see the file you redirected `stdout` and `stderr` to.

When a message `pbm agent is listening for the commands` is printed to the
`pbm-agent` log file, `pbm-agent` confirms that it has connected to its `mongod` node successfully.

## Next steps 

[Make a backup :material-arrow-right:](../usage/backup-physical.md){.md-button}


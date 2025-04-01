# Start the `pbm-agent` process

Start `pbm-agent` on every server with the `mongod` node installed. 

=== Using systemd

It is best to use the packaged service scripts to run `pbm-agent`.

```{.bash data-prompt="$"}
$ sudo systemctl start pbm-agent
$ sudo systemctl status pbm-agent
```

=== Manually

The following is an example of starting `pbm-agent` manually. The output is redirected to a file and the process is put in the background.

!!! important

    Start the `pbm-agent` as the `mongod` user. The `pbm-agent` requires write access to the MongoDB data directory to make physical restores.

```{.bash data-prompt="$"}
$ su mongod nohup pbm-agent --mongodb-uri "mongodb://username:password@localhost:27018/" > /data/mdb_node_xyz/pbm-agent.$(hostname -s).27018.log 2>&1 &
```

Replace `username` and `password` with those of your `pbm` user. `/data/mdb_node_xyz/` is the path where **pbm-agent** log files will be written. Make sure you have created this directory and granted write permissions to it for the `mongod` user.

Alternatively, you can run `pbm-agent` on a shell terminal temporarily if you want to observe and/or debug the startup from the log messages.

## Multiple agents on the same host

For example, if you put a config server (listen port `27019`) co-located on the same host as other `mongod` process (listen port `27018`). 
In this case there should be two `pbm-agent` processes, one connected to mongod process (e.g. `“mongodb://username:password@localhost:27018/”`) and one to the configsvr node (e.g. `“mongodb://username:password@localhost:27019/”`).

1. Prepare the URI for each agent
```
tee /etc/sysconfig/pbm-agent1 <<EOF
PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27017/?authSource=admin"
EOF
```
```
tee /etc/sysconfig/pbm-agent2 <<EOF
PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27018/?authSource=admin"
EOF
```

2. Prepare service files for each agent
```
tee /usr/lib/systemd/system/pbm-agent1.service <<EOF
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
```
tee /usr/lib/systemd/system/pbm-agent2.service <<EOF
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

3. Reload system units and start services
```
systemctl daemon-reload
systemctl start pbm-agent1
systemctl start pbm-agent2
```

## How to see the `pbm-agent` log

With the packaged `systemd` service, the log output to `stdout` is captured by
systemd’s default redirection to `systemd-journald`. You can view it with the
command below. See `man journalctl` for useful options such as `--lines`, `--follow`, etc.

```{.bash data-prompt="$"}
$ journalctl -u pbm-agent.service
-- Logs begin at Tue 2019-10-22 09:31:34 JST. --
Jan 22 15:59:14 : Started pbm-agent.
Jan 22 15:59:14 pbm-agent[3579]: pbm agent is listening for the commands
...
...
```

If you started `pbm-agent` manually, see the file you redirected stdout and stderr to.

When a message `pbm agent is listening for the commands` is printed to the
`pbm-agent` log file, `pbm-agent` confirms that it has connected to its `mongod` node successfully.

## Next steps 

[Make a backup :material-arrow-right:](../usage/backup-physical.md){.md-button}


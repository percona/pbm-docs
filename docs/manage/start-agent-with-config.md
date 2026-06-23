# Start pbm-agent using the configuration file

!!! admonition "Version added: [2.9.0](../release-notes/2.9.0.md)"

A `pbm-agent` requires a MongoDB connection string URI to start. You can define it either via the environment variable or use the configuration file. The latter option enables you to specify additional configuration like the number of parallel connections for backups and [custom logging configuration](logpath.md). 

This document explains how to start a `pbm-agent` using the configuration file. For how to use environment variables, see the [Configure authentication in MongoDB](../install/configure-authentication.md#set-the-mongodb-connection-uri-for-pbm-agent)

The steps vary slightly based on your installation method. In the following example, we provide all available configuration options for reference. Note that for initial `pbm-agent` start, `mongodb-uri` is mandatory.

## For installation from packages {.power-number}

1. Create a configuration file. For example, `/etc/pbm-agent.yaml`. 

	```yaml title="/etc/pbm-agent.yaml"
	mongodb-uri: mongodb://pbm:mysecret@localhost:27017/
	backup:
	  dump-parallel-collections: 10
	log:
	  path: "/var/log/pbm.json"
	  level: "I"
	  json: true
	```

2. Modify the `pbm-agent.service` systemd unit file. Specify the path to the configuration file for the `ExecStart` parameter. 

    ```init title="pbm-agent.service"
    ....

    [Service]
    ....
    ExecStart=/usr/bin/pbm-agent -f /etc/pbm-agent.yaml

    ....
    ```

3. Start the `pbm-agent`.

    ```bash
    sudo systemctl start pbm-agent
    ```

## For installation from the source code and tarballs {.power-number}

For the initial `pbm-agent` start, you require the credentials of a `pbm` user. See the [Create a pbm user](../install/configure-authentication.md#create-the-pbm-user) section for how to create this user. 

1. Create the configuration file. Specify the `pbm` user credentials in the MongoDB connection string URI.

	```yaml title="/etc/pbm-agent.yaml"
	mongodb-uri: mongodb://pbm:mysecret@localhost:27017/
	backup:
	  dump-parallel-collections: 10
	log:
	  path: "/var/log/pbm.json"
	  level: "I"
	  json: true
	```

2. Check that you have exported the location of the PBM binaries to the `$PATH` variable
3. Start the `pbm-agent`:

    ```bash
    pbm-agent -f /etc/pbm-agent.yaml
	```

See all available configuration options in the [pbm-agent configuration file options](../reference/pbm-agent-config-options.md) reference.

# Output pbm-agent logs to a file

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)"

By default, each `pbm-agent` writes its logs to the following paths:

* the `admin.pbmLogs` PBM Control collection which serves as the centralized logging storage
* `STDERR` stream on the each host where the agent is running

You can configure every `pbm-agent` to output logging information to a file at a custom path. This enhancement enables you to achieve the following:

* **optimize system logging** as the `pbm-agent` logs are stored in a separate file and don't mix with other system logs
* **introduce the log rotation policy** for the effective usage of the storage space 
* **simplify the log collection process** for further analysis
* **comply with the logging and auditing requirements** by storing logs in a secure location for the required period

## Logging configuration definition

You can define a logging configuration when you start a `pbm-agent` as follows:

* using the command line flags. This setting has the highest priority and overrides other setup options.
* using environment variables 
* [via the configuration file](start-agent-with-config.md). This setting is available starting with version 2.9.0.

## Logging configuration options

The logging configuration options are:

| Environment variable | Command line flag | Configuration file option |  Description | 
|----------------------|-------------------|-------------------|-------------|
| `LOG_PATH` | `--log-path` | `log.path` | The path to the log file. The file is created if it doesn't exist. The default value is `/dev/stderr` which means that the logs are written to the standard error output. |
| `LOG_LEVEL` | `--log-level` | `log.level`| The log severity level. Supported levels are (from low to high): D - Debug (default), I - Info, W - Warning, E - Error, F - Fatal.<br><br> The output includes both the specified severity level and all higher ones |
| `LOG_JSON`| `--log-json` | `log.json` | Output log messages in JSON format. If undefined, logs are written in the default text format. |

## Example

Start the `pbm-agent` on every node with the following command:


=== ":material-console: Command line"

	```bash
	pbm-agent --log-path=/var/log/pbm-agent.log --log-level=W --log-json
	```

=== ":material-variable: Environment variables"

	```bash
	export LOG_PATH=/var/log/pbm-agent.log
	export LOG_LEVEL=W
	export LOG_JSON=1
	pbm-agent
	```

=== ":octicons-file-code-24: Configuration file"

	1. Create a configuration file. For example, `/etc/pbm-agent.yaml`

    	```yaml title="/etc/pbm-agent.yaml"
        log:
           path: "/var/log/pbm.json"
           level: "I"
           json: true
    	```
    
    2. Start the `pbm-agent`:

        ```bash
		pbm-agent -f /etc/pbm-agent.yaml
		```


If PBM cannot write to the specified file due to some error, it falls back to the `STDERR` stream. And you can always retrieve the logging information using the `pbm logs` command.

This ability to output logs to a file at a custom path enhances log management, making it easier to monitor and audit PBM operations effectively, and facilitates regulatory compliance.

## Configuration reload

You can dynamically change the log level and output format via the configuration file. 

Update the desired options in the configuration file. PBM detects the changes and applies them saving you from restarting the agent.
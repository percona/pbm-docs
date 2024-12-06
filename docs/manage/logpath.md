# Output pbm-agent logs to a file

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)"


By default, each `pbm-agent` writes its logs to the following paths:

* the `admin.pbmLogs` PBM Control collection which serves as the centralized logging storage
* `STDERR` stream on the each host where the agent is running

Starting with version 2.8.0, you can configure every `pbm-agent` to output logging information to a file at a custom path. This enhancement enables you to achieve the following:

* optimize system logging as the `pbm-agent` logs are stored in a separate file and don't mix with other system logs
* introduce the log rotation policy for the effective usage of the storage space 
* simplify the log collection process for further analysis
* comply with the logging and auditing requirements by storing logs in a secure location for the required period

You can define a logging configuration either via the command line flags or via environment variables when you start a `pbm-agent`. The options are:

| Environment variable | Command line flag | Description | 
|----------------------|-------------------|-------------|
| `LOG_PATH` | `--log-path` | The path to the log file. The file is created if it doesn't exist. The default value is `/dev/stderr` which means that the logs are written to the standard error output. |
| `LOG_LEVEL` | `--log-level` | The log severity level. Supported levels are (from low to high): D - Debug (default), I - Info, W - Warning, E - Error, F - Fatal.<br><br> The output includes both the specified severity level and all higher ones |
| `LOG_JSON`| `--log-json` | Output log messages in JSON format. If undefined, logs are written in the default text format. |

## Example

Start the `pbm-agent` on every node with the following command:

=== ":material-variable: Environment variables"

	```{.bash data-prompt="$"}
	$ export LOG_PATH=/var/log/pbm-agent.log
	$ export LOG_LEVEL=W
	$ export LOG_json
	$ pbm-agent
	```

=== ":material-console: Command line"

	```{.bash data-prompt="$"}
	$ pbm-agent --log-path=/var/log/pbm-agent.log --log-level=W --log-json
	```

!!! note

    Command line flags take precedence over environment variables.

If PBM cannot write to the specified file due to some error, it falls back to `STDERR` stream. And you can always retrieve the logging information using the `pbm logs` command.

This ability to output logs to a file at a custom path enhances log management, making it easier to monitor and audit PBM operations effectively, and facilitates regulatory compliance.
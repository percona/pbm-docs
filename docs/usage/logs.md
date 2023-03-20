# View backup logs

!!! admonition "Version added: [1.4.0](../release-notes/1.4.0.md)"

You can see the logs from all `pbm-agents` in your MongoDB environment using `pbm CLI`. This reduces time for finding required information when troubleshooting issues.

!!! note 
    
    The log information about restores from physical backups is not available in pbm logs.

To view `pbm-agent` logs, run the `pbm logs` command and pass one or several flags to narrow down the search.

The following flags are available:

* `-t`, `--tail` - Show the last N rows of the log
* `-e`, `--event` - Filter logs by all backups or a specific backup
* `-n`, `--node` - Filter logs by a specific node  or a replica set
* `-s`, `--severity` - Filter logs by severity level. The following values are supported (from low to high):

    * `D` - Debug
    * `I` - Info
    * `W` - Warning
    * `E` - Error
    * `F` - Fatal

* `-o`, `--output` - Show log information as text (default) or in JSON format.
* `-i`, `--opid` - Filter logs by the operation ID

## Examples

The following are some examples of filtering logs:

**Show logs for all backups**

```{.bash data-prompt="$"}
$ pbm logs --event=backup
```

**Show the last 100 lines of the log about a specific backup 2020-10-15T17:42:54Z**

```{.bash data-prompt="$"}
$ pbm logs --tail=100 --event=backup/2020-10-15T17:42:54Z
```

**Include only errors from the specific replica set**

```{.bash data-prompt="$"}
$ pbm logs -n rs1 -s E
```

The output includes log messages of the specified severity type and all higher levels. Thus, when `ERROR` is specified, both `ERROR` and `FATAL` messages are shown in the output.

## Implementation details

`pbm-agents` write log information into the `pbmLog` collection in the [PBM Control collections](../reference/glossary.md#pbm-control-collections). Every `pbm-agent` also writes log information to `stderr` so that you can retrieve it when there is no healthy `mongod` node in your cluster or replica set. For how to view an individual `pbm-agent` log, see [How to see the pbm-agent log](../install/initial-setup.md#how-to-see-the-pbm-agent-log).

!!! note

    Log information from `pbmLog` collection is shown in the UTC timezone and from the stderr - in the server’s time zone.

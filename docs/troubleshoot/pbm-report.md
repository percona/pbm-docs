# Diagnostics report

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)"

When troubleshooting issues with backups and restores, viewing logs and PBM status may sometimes not be enough to identify the root of the issue. 

Starting with version 2.8.0 you can generate a diagnostics report about a specific backup, restore, or other commands. The report includes the following information:

* The information about the environment: pbm-agents statuses, cluster members, etc. 
* Logs collected between the start and end time of the command execution
* If it is a backup command, the backup metadata file.
* If it is a restore command, the restore metadata file and the backup metadata file. 

   <i warning>:material-alert: Warning:</i> Physical restore is not supported at the moment.

This data is stored in separate files in JSON format.

To generate a report, run the `pbm diagnostics` command:

```{.bash data-prompt="$"}
$ pbm diagnostic --path=path --name=<backup-name> 
```

or you can use the OPID of the command:

```{.bash data-prompt="$"}
$ pbm diagnostic --path=path --opid=<OPID> 
```

where:

* `path` is the path where to save the report. If the directory doesn’t exist, PBM creates it during the report generation. Make sure that the user that runs PBM CLI has write access to the specified path.
* `OPID` is the unique Operation ID of the specified command. You can retrieve it from the `pbm logs`, `pbm describe-backup` / `pbm describe-restore` output. 
* `name` is the name of the required backup or a restore. You can use it instead of OPID for backups and restores.

**Usage example**

For example, your `pbm status` output has the following backup:

```{.text .no-copy}
Backups:
========
S3 us-east-1 http://minio:9000/mybackups
  Snapshots:
    2024-11-27T13:49:31Z 95.79KB <logical> [restore_to_time: 2024-11-27T13:49:37Z]
```

To retrieve the OpID of the backup operation, run the `pbm describe-backup` command as follows:

```{.bash data-prompt="$"}
$ pbm describe-backup 2024-11-27T13:49:31Z | grep 'opid'
```

The output returns the OpID:

```{.text .no-copy}
opid: 6747236bfa98f6a85b9bd4e7
```

Now you can generate the diagnostics report:

```{.bash data-prompt="$"}
$ pbm diagnostic --path=/tmp/backup_report --opid=6747236bfa98f6a85b9bd4e7
```

Check the generated files:

```{.bash data-prompt="$"}
$ ls /tmp/backup_report
6747236bfa98f6a85b9bd4e7.backup.json  6747236bfa98f6a85b9bd4e7.log  6747236bfa98f6a85b9bd4e7.report.json
```

You can use the OPID to generate a diagnostics report about other operations like cleanup, cancellation, etc. In this case the report contains only the information about your environment and logs collected during the operation execution.

You can also output the report into an archive file as follows:

```{.bash data-prompt="$"}
$ pbm diagnostic --path=path --opid=<OPID> --archive
``` 

The diagnostics report empowers you to collect every necessary aspect for deep analysis of issues with a specific operation, all in one go. If you can't perform the analysis yourself, `pbm diagnostic` offers a quick and convenient way to collect and submit all relevant information for filing a bug report. This significantly reduces the interaction time between you and our experts, accelerating issue resolution.

Percona customers have the advantage of their bug reports being prioritized higher. If you're interested in enjoying these benefits, [contact us today :octicons-link-external-16:](https://www.percona.com/about/contact).

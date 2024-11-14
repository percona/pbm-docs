# Diagnostics report

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)"

When troubleshooting issues with backups and restores, viewing logs and PBM status may sometimes not be enough to identify the root of the issue. 

Starting with version 2.8.0 you can generate a diagnostics report about a specific *logical* backup or a restore. The report includes the following information:

* The information about the environment where a backup or a restore is running. 
* Logs collected between the start and end time of the command execution
* Metadata file of a specified backup or a restore. For a restore, includes metadata of this restore and the backup it was taken from. 

This data is stored in separate files in JSON format.

To generate a report, run the `pbm diagnostics` command:

```{.bash data-prompt="$"}
$ pbm diagnostic --path=path --name=<backup-name> 
```

or

```{.bash data-prompt="$"}
$ pbm diagnostic --path=path --opid=<OPID> 
```

where:

* `path` is the path where to save the report. If the directory doesn’t exist, PBM creates it during the report generation. Make sure that the user that runs PBM CLI has write access to the specified path.
* `name` is the name of the required backup or a restore.
* `OPID` is the unique Operation ID of the specified command. You can retrieve it from the `pbm describe-backup` / `pbm describe-restore` output. 

You can use the OPID to generate a diagnostics report about other operations like cleanup, cancellation, etc. In this case the report contains only the information about your environment and logs collected during the operation execution.

You can also output the report into an archive file. 

The diagnostics report empowers you to collect every necessary aspect for deep analysis of issues with a specific operation, all in one go. If you can't perform the analysis yourself, `pbm diagnostic` offers a quick and convenient way to collect and submit all relevant information for filing a bug report. This significantly reduces the interaction time between you and our experts, accelerating issue resolution.

Percona customers have the advantage of their bug reports being prioritized higher. If you're interested in enjoying these benefits, [contact us today :octicons-link-external-16:](https://www.percona.com/about/contact).

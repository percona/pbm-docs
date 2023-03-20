# View restore progress

!!! admonition "Version added: [2.0.0](../release-notes/2.0.0.md)"

You can track the status of both physical and logical restores. This gives you a clear understanding of the restore progress so that you can react accordingly. 

To view the restore status, run the `pbm describe-restore` command and specify the restore name. To track the progress of a physical restore, also specify the path to the Percona Backup for MongoDB configuration file. Since `mongod` nodes are shut down during a physical restore, Percona Backup for MongoDB uses the configuration file to read the restore status on storage.

```{.bash data-prompt="$"}
$ pbm describe-restore 2022-08-15T11:14:55.683148162Z -c pbm_config.yaml
```

The output provides the following information:

-  Restore name
-  The name of the backup from which the database was restored
-  Type
-  Status
-  opID
-  The time of the restore start
-  Last transition time – the time when the restore process changed its status
-  The name of every replica set, its restore status and the last transition time 

For physical backups only, the following additional information is provided:

- The node name
- Restore status on the node
- Last transition time

For version 1.8.1 and earlier, tracking restore progress during physical restores is not available. To check the restore status, the options are:

- Check the `stderr` logs of the leader `pbm-agent`. The leader ID is printed once the restore has started.
- Check the status in the metadata file created on the remote storage for the restore. This file is in the root of the storage path and has the format `.pbm.restore/<restore_timestamp>.json`.
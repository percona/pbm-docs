> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# View restore progress

Version added: [2.0.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.0.0.html)

You can track the status of both physical and logical restores. This gives you a clear understanding of the restore progress so that you can react accordingly.

To view the restore status, run the `pbm describe-restore` command and specify the restore name. To track a physical restore progress, specify the path to the Percona Backup for MongoDB (PBM) configuration file. Since `mongod` nodes are shut down during a physical restore, Percona Backup for MongoDB uses the configuration file to read the restore status on storage. PBM stores the restore metadata only in the main storage when [multiple backup storages](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/multi-storage.html) are configured.

```
pbm describe-restore 2022-08-15T11:14:55.683148162Z -c pbm_config.yaml
```

The output provides the following information:

- Restore name
- The name of the backup from which the database was restored
- Type
- Status
- opID
- The time of the restoration start
- The time of the restore finish (for successful restores)
- Last transition time – the time when the restore process changed its status
- The name of every replica set, its restore status, and the last transition time

For physical backups only, the following additional information is provided:

- The node name
- Restore status on the node
- Last transition time

Check the [pbm describe-restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#output_1) for the full list of fields and their description.

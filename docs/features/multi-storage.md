# Multiple storages for backups

!!! admonition "Version added: [](../release-notes/.md)"

A good practice in a backup policy is to follow the 3-to-1 rule: have 3 copies of data stored on 2 different storages and have 1 copy kept offsite. Instead of transferring data to multiple storages, you can configure those storages and have PBM make backups to them directly, at different schedules. 

For example, you can configure backups as follows:

* Make full physical backups with point-in-time recovery enabled weekly and store them on AWS S3 bucket, 
* Make incremental backups daily on another storage.
* Make EBS snapshots of the whole database monthly and store it on an external offsite server.

This ability to define multiple storages for backups brings the following benefits:

* Saves costs on data transfer in case of cloud storages
* Increases effectiveness of following your organization’s backup policy either via your own applications and tools interfaced with PBM or via Percona Everest

## Configuration profiles 

By default, PBM stores backups and point-in-time recovery oplog slices to the remote backup storage which you defined in the configuration file during the initial setup. This is the **main** backup storage.

To make backups to additional – **external** backup storages, a concept of a configuration profile is introduced. A configuration profile is a file that stores only the configuration for an external backup storage.

Here’s the example of the configuration profile:

```yaml title="minio.yaml"
storage:
  type: s3
  s3:
	endpointUrl: "http://localhost:9000"
	bucket: mybucket
	region: my-region
	credentials:
		access_key_id: myaccesskey
		secret_access_key: mysecretkey
	
```

To upload the configuration profile to PBM, use the [`pbm profile add`](../reference/pbm-commands.md#pbm-profile-add) command and specify the path to the profile.

```{.bash data-prompt="$"}
$ pbm profile add <profile_name> /path/to/profile.yaml
```

To show the information about the external backup storage, use the [`pbm profile show`](../reference/pbm-commands.md#pbm-profile-show) command:

```{.bash data-prompt="$"}
$ pbm profile show <profile_name>
```

See the full list of the configuration profile management commands in the [pbm commands](../reference/pbm-commands.md) reference.

## Make a backup to an external storage

To make a backup to an external backup storage, pass the profile name with the `--profile` flag for the `pbm backup` command. For example, to run a physical backup and store it in the MinIO storage defined via the `minio` configuration profile, run the following command:

```{.bash data-prompt="$"}
$ pbm backup -t physical --profile=minio --wait 
```

??? example "Sample output"

    {.text .no-copy}
    Starting backup '2024-06-25T11:25:30Z'....Backup '2024-06-25T11:25:30Z' to remote store 's3://http://minio:9000/backup' has started

You can make any type of backups except snapshot-based ones on an external storage.

Note that point-in-time recovery oplog slicing is not stopped automatically for backups made to an external storage. Thus, PBM saves oplog chunks related to such backups on both the main and the external storages.

When you make incremental backups, make sure to keep the whole backup chain on the same storage. To switch the backup storage, you must make a new base backup on it to start the new incremental chain. 

## Restore from an external storage

Before you start, make sure that `pbm-agents` have the read permissions to backups on the remote storage(s). Also, [make all preparations for the restore](../usage/restore.md#before-you-start).

1. List backups by running the `pbm list` or `pbm status` commands.
    
    ```{.bash data-prompt="$"}
	$ pbm list
	```

	The output shows the backup names and timestamps. External backups are marked with an asterisk:

	??? example "Sample output"

	    ```{.text .no-copy}
	    Backup snapshots:
	      2024-06-25T10:53:57Z <logical> [restore_to_time: 2024-06-25T10:54:02Z]
	      2024-06-25T10:54:55Z <logical, *> [restore_to_time: 2024-06-25T10:55:02Z]
	      2024-06-25T10:57:49Z <logical, *> [restore_to_time: 2024-06-25T10:57:56Z]

	    PITR <on>:
	      2024-06-25T10:54:03Z - 2024-06-25T10:57:51Z
	    ```

2. To make a point-in-time restore, you must explicitly pass the backup name for the `pbm restore` command:

    ```{.bash data-prompt="$"}
    $ pbm-restore --time=<timestamp> --base-snapshot <backup-name>
    ```

3. After the restore is complete, do the required post-restore steps depending on the restore type.
4. Make a fresh backup to serve as the new base for future restores. 

## Delete backups

You can delete backups from an external storage only by name. 

Run the `pbm delete` command and pass the backup name:

```{.bash data-prompt="$"}
$ pbm delete-backup 2024-06-25T10:54:55Z
```

## Implementation specifics

1. You can make backups of any type except snapshot-based ones on the external storage.
2. To start point-in-time recovery oplog slicing, you must make a backup on the main storage. A backup from an external storage is not considered a valid base backup for oplog slicing.
3. PBM saves point-in-time recovery oplog ranges only on the main storage. Backups are saved on the storage that you define when starting a backup. 
4. Backup process on the external storage doesn’t stop point-in-time recovery oplog slicing on the main storage. Thus, PBM saves oplog chunks related to such backups on both the main and the external storages
5. The whole incremental chain must be stored on the same storage. To change the storage for incremental backups, you must start a new backup chain with the incremental base backup on the new storage.
6. To restore from a backup on external storage, pbm-agents must have read permissions on it.
7. To make a point-in-time recovery, you must specify the backup name via the `--base-snapshot` flag. Without it, PBM searches for the base backup on the main storage.
8. You can delete backups from external storages only by name using the `pbm delete-backup <backup-name>` command. Bulk deletion of backups older than the specified time is supported only for the main storage.




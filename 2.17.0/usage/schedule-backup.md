> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Schedule backups

We recommend using `crond` or similar services to schedule backup snapshots.

Important

Before configuring `crond`, make sure that you have [installed](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) and [configured](https://docs.percona.com/percona-backup-mongodb/2.17.0/install/initial-setup.html) Percona Backup for MongoDB to make backups in your database. Start a backup manually to verify this:

```
pbm backup
```

The recommended approach is to create a `crontab` file in the `/etc/cron.d` directory and specify the command in it. This simplifies server administration especially if multiple users have access to it.

`pbm` CLI requires a [valid MongoDB URI connection string](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/authentication.html) to authenticate in MongoDB. Instead of specifying the MongoDB URI connection string as a command line argument, which is a potential security risk, we recommend creating an environment file and specify the `export PBM_MONGODB_URI=$PBM_MONGODB_URI` statement within.

As an example, let’s configure to run backup snapshots on 23:30 every Sunday. The steps are the following:

1. Create an environment file. Let’s name it `pbm-cron`.

   **Debian and Ubuntu**

   ```
   vim /etc/default/pbm-cron
   ```

   **Red Hat Enterprise Linux and derivatives**

   ```
   vim /etc/sysconfig/pbm-cron
   ```

1. Specify the environment variable in `pbm-cron`:

   ```
   export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?replSetName=xxxx"
   ```

1. Grant access to the `pbm-cron` file for the user that will execute the `cron` task.

1. Create a `crontab` file. Let’s name it `pbm-backup`.

   ```
   touch pbm-backup
   ```

1. Specify the command in the file:

   ```
   30 23 * * sun <user-to-execute-cron-task> . /etc/default/pbm-cron; /usr/bin/pbm backup
   ```

   Note the dot `.` before the environment file. It sources (includes) the environment file for the rest of the shell commands.

1. Verify that backups are running in `/var/log/cron` or `/var/log/syslog` logs:

   ```
   grep CRON /var/log/syslog
   ```

## Schedule backups with point-in-time recovery running

It is convenient to automate making backups on a schedule using `crond` if you enabled [point-in-time recovery](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/point-in-time-recovery.html).

You can configure point-in-time recovery and `crond` in any order. Note, however, that point-in-time recovery will start running only after at least one full backup has been made.

- Make a fresh backup manually. It will serve as the starting point for incremental backups.
- Enable point-in-time recovery.
- Configure `crond` to run backup snapshots on a schedule.

When it is time for another backup snapshot, Percona Backup for MongoDB automatically disables point-in-time recovery and re-enables it once the backup is complete.

## Backup storage cleanup

Previous backups are not automatically removed from the backup storage. You need to remove the oldest ones periodically to limit the amount of space used in the backup storage.

Version added: [2.1.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.1.0.html)

Starting with version 2.1.0, you can use the [`pbm cleanup --older-than`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-cleanup) command to delete outdated backup snapshots and point-in-time recovery oplog slices. Learn more about how PBM deletes outdated data in the [Clean up outdated data](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/delete-backup.html#delete-outdated-data) section.

You can configure a `cron` task to automate storage cleanup by specifying the following command in the `crontab` file:

```
/usr/bin/pbm cleanup -y --older-than 30d --wait
```

This command deletes backups and oplog slices that are older than 30 days. PBM automatically updates the metadata without requiring a manual resync, simplifying automation workflows.

You can change the period by specifying a desired interval for the `--older-than` flag.

Starting with version 2.13.0, you can also clean up backups from [external storages](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/multi-storage.html) by specifying both the `--older-than` and the `--profile` flags:

```
/usr/bin/pbm cleanup -y --older-than 30d --profile=minio --wait
```

This approach streamlines your retention policy across multiple storages and reduces operational overhead, eliminating the need for custom cleanup scripts.

For PBM version 2.0.5 and earlier, use the [`pbm delete backup --older-than <timestamp>`](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html#pbm-delete-backup) command. You can configure a `cron` task to automate backup deletion by specifying the following command in the `crontab` file:

```
/usr/bin/pbm delete-backup -f --older-than $(date -d '-1 month' +\%Y-\%m-\%d)
```

This command deletes backups that are older than 30 days. You can change the period by specifying a desired interval for the `date` function.

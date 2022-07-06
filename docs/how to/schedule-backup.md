# Schedule backups

In Percona Backup for MongoDB version 1.4.1 and earlier, we recommend using `crond` or similar services to schedule backup snapshots.

!!! important

    Before configuring `crond`, make sure that you have [installed](../installation.md) and [configured](../initial-setup.md) Percona Backup for MongoDB to make backups in your database. Start a backup manually to verify this: 

    ```sh
    pbm backup
    ```

The recommended approach is to create a `crontab` file in the `/etc/cron.d` directory and specify the command in it. This simplifies server administration especially if multiple users have access to it.

`pbm` CLI requires a [valid MongoDB URI connection string](../details/authentication.md) to authenticate in MongoDB. Instead of specifying the MongoDB URI connection string as a command line argument, which is a potential security risk, we recommend creating an environment file and specify the `export PBM_MONGODB_URI=$PBM_MONGODB_URI` statement within.

As an example, let’s configure to run backup snapshots on 23:30 every Sunday.
The steps are the following:


1. Create an environment file. Let’s name it `pbm-cron`.


    === "Debian and Ubuntu"

        ```sh
        vim /etc/default/pbm-cron
        ``` 

    === "Red Hat Enterprise Linux and derivatives"

        ```sh
        vim /etc/sysconfig/pbm-cron
        ```


2. Specify the environment variable in `pbm-cron`:
    
     ```sh
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27018?/replSetName=xxxx"
     ```

3. Grant access to the `pbm-cron` file for the user that will execute the `cron` task.


4. Create a `crontab` file. Let’s name it `pbm-backup`.

     ```sh
     touch pbm-backup
     ```

5. Specify the command in the file:

     ```sh
     30 23 * * sun <user-to-execute-cron-task> . /etc/default/pbm-cron; /usr/bin/pbm backup
     ```

    !!! note "" 
     
        Note the dot `.` before the environment file. It sources (includes) the environment file for the rest of the shell commands.
 

6. Verify that backups are running in `/var/log/cron` or `/var/log/syslog` logs:

     ```sh
     grep CRON /var/log/syslog
     ```

## Schedule backups with Point-in-Time Recovery running

It is convenient to automate making backups on a schedule using `crond` if you enabled [Point-in-Time Recovery](../usage/point-in-time-recovery.md).

You can configure Point-in-Time Recovery and `crond` in any order. Note, however, that Point-in-Time Recovery will only start running after at least one full backup has been made.

 * Make a fresh backup manually. It will serve as the starting point for incremental backups
 * Enable point-in-time recovery
 * Configure `crond` to run backup snapshots on a schedule

When it is time for another backup snapshot, Percona Backup for MongoDB automatically disables Point-in-Time Recovery and re-enables it once the backup is complete.

## Backup storage cleanup

Previous backups are not automatically removed from the backup storage. You need to remove the oldest ones periodically to limit the amount of space used in the backup storage.

We recommend using the [`pbm delete backup --older-than <timestamp>`](../reference/pbm-commands.md#pbm-delete-backup) command. You can configure a `cron` task to automate backup deletion by specifying the following command in the `crontab` file:

```sh
/usr/bin/pbm delete-backup -f --older-than $(date -d '-1 month' +\%Y-\%m-\%d)
```

This command deletes backups that are older than 30 days. You can change the period by specifying a desired interval for the `date` function.

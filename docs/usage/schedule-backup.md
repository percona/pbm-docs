# Schedule backups

We recommend using `crond` or similar services to schedule backup snapshots.

!!! important

    Before configuring `crond`, make sure that you have [installed](../installation.md) and [configured](../install/initial-setup.md) Percona Backup for MongoDB to make backups in your database. Start a backup manually to verify this: 

    ```{.bash data-prompt="$"}
    $ pbm backup
    ```

The recommended approach is to create a `crontab` file in the `/etc/cron.d` directory and specify the command in it. This simplifies server administration especially if multiple users have access to it.

`pbm` CLI requires a [valid MongoDB URI connection string](../details/authentication.md) to authenticate in MongoDB. Instead of specifying the MongoDB URI connection string as a command line argument, which is a potential security risk, we recommend creating an environment file and specify the `export PBM_MONGODB_URI=$PBM_MONGODB_URI` statement within.

As an example, let’s configure to run backup snapshots on 23:30 every Sunday.
The steps are the following:


1. Create an environment file. Let’s name it `pbm-cron`.


    === "Debian and Ubuntu"

        ```{.bash data-prompt="$"}
        $ vim /etc/default/pbm-cron
        ``` 

    === "Red Hat Enterprise Linux and derivatives"

        ```{.bash data-prompt="$"}
        $ vim /etc/sysconfig/pbm-cron
        ```


2. Specify the environment variable in `pbm-cron`:
    
     ```{.bash data-prompt="$"}
     $ export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017?/replSetName=xxxx"
     ```

3. Grant access to the `pbm-cron` file for the user that will execute the `cron` task.


4. Create a `crontab` file. Let’s name it `pbm-backup`.

     ```{.bash data-prompt="$"}
     $ touch pbm-backup
     ```

5. Specify the command in the file:

     ```{.bash data-prompt="$"}
     30 23 * * sun <user-to-execute-cron-task> . /etc/default/pbm-cron; /usr/bin/pbm backup
     ```

    !!! note "" 
     
        Note the dot `.` before the environment file. It sources (includes) the environment file for the rest of the shell commands.
 

6. Verify that backups are running in `/var/log/cron` or `/var/log/syslog` logs:

     ```{.bash data-prompt="$"}
     $ grep CRON /var/log/syslog
     ```

## Schedule backups with point-in-time recovery running

It is convenient to automate making backups on a schedule using `crond` if you enabled [point-in-time recovery](../features/point-in-time-recovery.md).

You can configure point-in-time recovery and `crond` in any order. Note, however, that point-in-time recovery will start running only after at least one full backup has been made.

 * Make a fresh backup manually. It will serve as the starting point for incremental backups.
 * Enable point-in-time recovery.
 * Configure `crond` to run backup snapshots on a schedule.

When it is time for another backup snapshot, Percona Backup for MongoDB automatically disables point-in-time recovery and re-enables it once the backup is complete.

## Backup storage cleanup

Previous backups are not automatically removed from the backup storage. You need to remove the oldest ones periodically to limit the amount of space used in the backup storage.

!!! admonition "Version added: [2.1.0](../release-notes/2.1.0.md)"

Starting with version 2.1.0, you can use the `pbm cleanup --older-than` command to delete outdated backup snapshots and point-in-time recovery oplog slices. You can configure a `cron` task to automate storage cleanup by specifying the following command in the `crontab` file:

```{.bash data-prompt="$"}
$ $ /usr/bin/pbm cleanup -y --older-than 30d --wait
``` 

This command deletes backups and oplog slices that are older than 30 days. You can change the period by specifying a desired interval for the `--older-than` flag. 


!!! admonition ""

    For PBM version 2.0.5 and earlier, use the [`pbm delete backup --older-than <timestamp>`](../reference/pbm-commands.md#pbm-delete-backup) command. You can configure a `cron` task to automate backup deletion by specifying the following command in the `crontab` file:

    ```{.bash data-prompt="$"}
    $ /usr/bin/pbm delete-backup -f --older-than $(date -d '-1 month' +\%Y-\%m-\%d) 
    ```

    This command deletes backups that are older than 30 days. You can change the period by specifying a desired interval for the `date` function.

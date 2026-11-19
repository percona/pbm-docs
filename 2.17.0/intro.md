> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# How Percona Backup for MongoDB works

Even in a highly-available architecture, such as with MongoDB replication, backups are still required even though losing one server is not fatal. Be it a complete or partial data disaster, you can use PBM (Percona Backup for MongoDB) to go back in time to the best available backup snapshot.

Percona Backup for MongoDB is a command line interface tool. It provides [the set of commands](https://docs.percona.com/percona-backup-mongodb/2.17.0/reference/pbm-commands.html) to manage backup and restore operations in your database.

[What backup to choose?](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)

## Usage example

Let’s have a look at how Percona Backup for MongoDB works.

With [Percona Backup for MongoDB up and running](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html) in your environment, make a backup:

```
pbm backup --type=logical
```

To save all events that occurred to the data between backups, enable the point-in-time recovery routine that saves oplog slices:

```
pbm config --set pitr.enabled=true
```

Now, imagine that your web application’s update was released on February 7 at 03:00 UTC. By 15:23 UTC, someone realizes that this update has a bug that is wiping the historical data of any user who logged in. To remediate this negative impact on data, it’s time to roll back up to the time of the application’s update - up to February 7, 03:00 UTC.

```
pbm list
```

Sample output

```
Backup snapshots:
    2024-02-05T13:55:55Z [complete: 2024-02-05T13:56:15]
    2024-02-07T13:57:58Z [complete: 2024-02-07T13:58:17]
    2024-02-03T08:08:15Z [complete: 2024-02-03T08:08:35]
    2024-02-09T14:06:06Z [complete: 2024-02-09T14:06:26]
    2024-02-11T14:22:41Z [complete: 2024-02-11T14:23:01]
```

The output lists the valid time ranges for the restore. The desired time (February 7, 03:00 UTC) falls within the `2024-02-03T08:08:36Z-2024-02-09T12:20:23Z` range, so let’s restore the database up to that time.

```
pbm restore --time 2024-02-07T02:59:59
```

To be on the safe side, it is a good practice to make a fresh backup after the restore is complete.

```
pbm backup
```

This backup refreshes the timeline and serves as the base for saving oplog slices. The point-in-time recovery routine is re-enabled automatically. It copies the slices taken during the backup and continues oplog slicing from the latest timestamp to ensure oplog continuity.

## Next steps

Ready to try it out?

[Quickstart](https://docs.percona.com/percona-backup-mongodb/2.17.0/installation.html)

## Useful links

- [PBM architecture](https://docs.percona.com/percona-backup-mongodb/2.17.0/details/architecture.html)
- [Backup types](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/backup-types.html)

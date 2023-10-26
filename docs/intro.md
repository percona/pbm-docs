# How Percona Backup for MongoDB works

Even in a highly-available architecture, such as with MongoDB replication, backups are still required even though losing one server is not fatal. Whether for a complete or partial data disaster, you can use PBM (Percona Backup for MongoDB) to go back in time to the best available backup snapshot.

Percona Backup for MongoDB is a command line interface. It provides [the set of commands](reference/pbm-commands.md) to manage backup and restore operations in your database.

## Usage example

Let's have a look at how Percona Backup for MongoDB works.

With [Percona Backup for MongoDB up and running](installation.md) in your environment, make a backup:

```{.bash data-prompt="$"}
$ pbm backup
```

To save all events that occurred to the data between the backups, enable saving oplog slices:

```{.bash data-prompt="$"}
$ pbm config --set pitr.enabled=true
```

Now, imagine that your web application’s update was released on February 7 at 03:00 UTC. By 15:23 UTC, someone realizes that this update has a bug that is wiping the historical data of any user who logged in. To remediate this negative impact on data, it’s time to roll back up to the time of the application’s update - up to February 7, 03:00 UTC.

```{.bash data-prompt="$"}
$ pbm list
```

The output lists the valid time ranges for the restore. The desired time (February 7, 03:00 UTC) falls within the `2021-02-03T08:08:36Z-2021-02-09T12:20:23Z` range, so let’s restore the database up to that time.

Since the restore and saving oplog slices are exclusive operations and cannot run together, let’s stop the oplog slicing first:

```{.bash data-prompt="$"}
$ pbm config --set pitr.enabled=false
```

Now, let's restore the database:

```{.bash data-prompt="$"}
$ pbm restore --time 2021-02-07T02:59:59
```

To be on the safe side, it is a good practice to make a fresh backup after the restore is complete.

```{.bash data-prompt="$"}
$ pbm backup
```

This backup refreshes the timeline and serves as the base for saving oplog slices. To re-enable this process, run:

```{.bash data-prompt="$"}
$ pbm config --set pitr.enabled=true
```

## Next steps

Ready to try it out? 

[Quickstart](installation.md){.md-button}

## Useful links

* [PBM architecture](details/architecture.md)
* [Backup types](features/backup-types.md)

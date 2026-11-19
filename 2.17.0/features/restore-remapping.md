> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Restoring into a cluster / replica set with a different name

Starting with version [1.8.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/1.8.0.html), you can restore [logical backups](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/logical.html) into a new environment that has the same or more number of shards and these shards have different replica set names. Starting with version [2.2.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.2.0.html), you can restore environments that have [custom shard names](https://www.mongodb.com/docs/manual/reference/command/addShard/#mongodb-dbcommand-dbcmd.addShard).

Starting with version [2.2.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.2.0.html), you can restore [physical](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/physical.html) and [incremental physical](https://docs.percona.com/percona-backup-mongodb/2.17.0/features/incremental-backup.html) backups into a new environment with a different replica set names. Note that **the number of shards must be the same** as in the environment where the you made the backup.

To restore data to the environment with different replica set names, configure the name mapping between the target and source environments. You can either set the `PBM_REPLSET_REMAPPING` environment variable for `pbm` CLI or use the `--replset-remapping` flag for PBM commands. The mapping format is `<rsTarget>=<rsSource>`.

Important

Configure replica set name mapping for all shards in your cluster. Otherwise, Percona Backup for MongoDB attempts to restore the unspecified shard to the target shard with the same name. If there is no shard with such name or it is already mapped to another source shard, the restore fails.

Configure the replica set name mapping:

**Using the environment variable for pbm CLI in your shell**

```
export PBM_REPLSET_REMAPPING="rsTarget1=rsSource1,rsTarget2=rsSource2"
```

Let’s say your source replica sets are `rsA` and `rsB` while the target ones are `rsX` and `rsY`. Then the command to export the environment variable is the following:

```
export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"
```

**Using the command line**

```
pbm restore <timestamp> --replset-remapping="rsTarget1=rsSource1,rsTarget2=rsSource2"
```

Let’s say your source replica sets are `rsA` and `rsB` while the target ones are `rsX` and `rsY`. Then the command to run a restore is the following:

```
pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"
```

The `--replset-remapping` flag is available for the following commands: `pbm restore`, `pbm list`, `pbm status`, `pbm oplog-replay`.

Version added: [2.14.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.14.0.html)

Before a restore operation is executed you have to confirm the action (to bypass it, add the `-y` or `--yes` flag).

Note

Follow the [post-restore steps](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore.html#post-restore-steps) on the new environment after the restore is complete.

This ability to restore data to clusters with different replica set names and the number of shards extends the set of environments compatible for the restore.

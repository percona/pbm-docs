# Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)

The configuration information is stored in a single document of the `admin.pbmConfig` collection. That single copy is shared by all the `pbm-agent` processes in a cluster (or non-sharded replica set), and can be read or updated using the `pbm` CLI tool.

You can see the whole config by running:

```{.javascript  data-prompt=">"}
> db.getSiblingDB(“admin”).pbmConfig.findOne()
```

But you don’t have to use the `mongo` shell; the `pbm` CLI has a “config” subcommand to read and update it.

Percona Backup for MongoDB config contains the following settings:

* [Remote backup storage configuration](configuration-options.md) is available starting with version 1.0 or 1.1.

* [Point-in-time recovery configuration](pitr-options.md) is available starting with version 1.3.0.

* [Restore options](restore-options.md) are available as starting with version 1.3.2.

* [Logging options](logging-options.md) are available starting with version 2.9.0.


Run [`pbm config --list`](../reference/pbm-commands.md#pbm-config) to see the whole config. Sensitive fields such as keys will be redacted.

## Insert the whole Percona Backup for MongoDB config from a YAML file

If you are initializing a cluster or a non-sharded replica set for the first time, it is simplest to write the whole config as a YAML file and use the
`pbm config --file` command to upload all the values in one command.

A config file must have a remote backup storage configuration. Find the config file examples for every supported remote backup storage in the Storage section. For more information about available config file options, see [Configuration file options](configuration-options.md).

Use the following command to upload the config file. For example, the config file name is `pbm_config.yaml`:

```{.bash data-prompt="$"}
$ pbm config --file pbm_config.yaml
```

Execute the command while connecting to the config server replica set if it is a
cluster. Otherwise, connect to the non-sharded replica set as normal. (See
[MongoDB connection strings - A Reminder (or Primer)](../details/authentication.md) if you are not familiar with MongoDB connection strings yet.)

## Accessing or updating single config values

You can set a single value at a time. For nested values, use dot-concatenated key names as shown in the following example:

```sh
pbm config --set storage.s3.bucket="operator-testing"
```

You can specify just the key name to list a single value.  If set, the command returns the value.

=== "Success"

    ```sh
    pbm config storage.s3.bucket
    operator-testing
    ```

=== "No value"

    ```sh
    pbm config storage.s3.INVALID-KEY
    Error: unable to get config key: invalid config key
    ``` 

## Synchronize configuration

When you upload a configuration file to PBM either during the initial setup or after you make changes, PBM automatically detects whether it needs to update the local metadata about backups, restores, and point-in-time recovery chunks  in PBM Control collections. 

For example, if you change the storage configuration, the metadata also changes, and PBM imports it from the storage. But if you only enabled/disabled point-in-time recovery, the metadata remains unchanged.

Though PBM synchronizes metadata automatically, there are cases when you need to run a manual synchronization:

* When you made changes to the storage manually. For example, you manually added a backup or changed the path to backups.
* As a post-restore step after a physical restore. After the data is copied back to the `mongod` nodes, you must manually trigger metadata synchronization from the backup storage.  

To sync the metadata, run the following command in the cluster/replica set:

```{.bash data-prompt="$"}
$ pbm config --force-resync
```

# Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)

The configuration information is stored in a single document of the `admin.pbmConfig` collection. That single copy is shared by all the `pbm-agent` processes in a cluster (or non-sharded replica set), and can be read or updated using the `pbm` CLI tool.

You can see the whole config by running

```{.javascript  data-prompt=">"}
> db.getSiblingDB(“admin”).pbmConfig.findOne()
```

But you don’t have to use the `mongo` shell; the `pbm` CLI has a “config” subcommand to read and update it.

Percona Backup for MongoDB config contains the following settings:

* [Remote backup storage configuration](../details/storage-configuration.md) is available starting with version 1.0 or 1.1.

* [Point-in-time recovery configuration](../features/point-in-time-recovery.md) is available starting with version 1.3.0.

* [Restore options](configuration-options.md#restore-options) are available as starting with version 1.3.2.

Run [`pbm config --list`](../reference/pbm-commands.md#pbm-config) to see the whole config. (Sensitive fields such as keys will be redacted.)

## Insert the whole Percona Backup for MongoDB config from a YAML file

If you are initializing a cluster or a non-sharded replica set for the first time, it is simplest to write the whole config as YAML file and use the
`pbm config --file` command to upload all the values in one command.

Find the config file examples for the remote backup storage (required) in the [Example config files](../details/storage-config-example.md) section. For more information about available config file options, see [Configuration file options](configuration-options.md#remote-backup=storage-options).

Use the following command to upload the config file. For example, config file name is `pbm_config.yaml`:

```{.bash data-prompt="$"}
$ pbm config --file pbm_config.yaml
```

Execute the command while connecting to config server replica set if it is a
cluster. Otherwise just connect to the non-sharded replica set as normal. (See
[MongoDB connection strings - A Reminder (or Primer)](../details/authentication.md) if you are not familiar with MongoDB connection strings yet.)

## Accessing or updating single config values

You can set a single value at a time. For nested values, use dot-concatenated key names as shown in the following example:

```sh
pbm config --set storage.s3.bucket="operator-testing"
```

To list a single value, you can specify just the key name by itself.  If set, the command returns the value.

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

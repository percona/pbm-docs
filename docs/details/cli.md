# PBM Command Line Utility (`pbm`)

`pbm` CLI is the command line tool with which you operate Percona Backup for MongoDB. `pbm` provides the **pbm** command that you will use manually in the shell. It will also
work as a command that can be executed in scripts (for example, by `crond`).

The set of [pbm sub-commands](../reference/pbm-commands.md) enables you to manage backups in your MongoDB environment.

`pbm` uses [PBM Control collections](control-collections.md) to communicate with `pbm-agent` processes. It starts and monitors backup or restore operations by updating and reading the corresponding PBM control collections for operations, log, etc. Likewise, it modifies the PBM config by saving it in the PBM Control collection for config values.

`pbm` CLI does not have its own config and/or cache files. Setting the
`PBM_MONGODB_URI` environment variable in your shell is a
configuration-like step that should be done for practical ease though. (Without
`PBM_MONGODB_URI`, the `--mongodb-uri` command line argument will need to
be specified each time.)

To learn how to set the `PBM_MONGODB_URI` environment variable, see [Set the MongoDB connection URI for `pbm` CLI](../install/initial-setup.md#set-mongodburi-pbm-cli). For more information about MongoDB URI connection strings, see [Authentication](authentication.md).
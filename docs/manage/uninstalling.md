# Uninstall Percona Backup for MongoDB

To uninstall Percona Backup for MongoDB, do the following steps:


1. Check that no backups are currently in progress in the output of [`pbm list`](../reference/pbm-commands.md#pbm-list).

2. Before the next 2 steps, make sure you know where the remote backup storage
is, so you can delete backups made by Percona Backup for MongoDB. If it is an  S3-compatible object storage, you will need to use another tool such as Amazon AWS’s “aws s3”, Minio’s `mc`, the web AWS Management Console, etc. to do that once Percona Backup for MongoDB is uninstalled. Don’t forget to note the connection credentials before they are deleted too.

3. Uninstall the **pbm-agent** and `pbm` executables. If you installed using a
package manager, see [Install Percona Backup for MongoDB](../installation.md) for relevant package names and commands for your OS distribution.

4. Drop the PBM control collections.

5. Drop the PBM database user. If this is a cluster, the `dropUser` command will
need to be run on each shard as well as in the config server replica set.

6. (Optional) Delete the backups from the remote backup storage.

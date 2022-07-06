# Authentication

Percona Backup for MongoDB has no authentication and authorization subsystem of its own - it uses the one of MongoDB. This means that `pbm` CLI and `pbm-agent` only require a valid MongoDB connection URI string for the `pbm` user.

For the S3-compatible remote storage authentication config, see
[Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](../reference/config.md).

## MongoDB connection strings - A Reminder (or Primer)

Percona Backup for MongoDB uses [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) strings to open
MongoDB connections. Neither `pbm` nor `pbm-agent` accept legacy-style
command-line arguments for `--host`, `--port`, `--user`, `--password`,
etc. as the `mongo` shell or `mongodump` command does.

```sh
$ pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27018/?authSource=admin"
$ #Alternatively:
$ export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27018/?authSource=admin"
$ pbm-agent
```

```sh
$ pbm list --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27018,mongocsvr2:27018,mongocsvr3:27018/?replicaSet=configrs&authSource=admin"
$ #Alternatively:
$ export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27018,mongocsvr2:27018,mongocsvr3:27018/?replicaSet=configrs&authSource=admin"
$ pbm list
```

The connection URI above is the format that MongoDB drivers accept universally
since approximately the release time of MongoDB server v3.6. The `mongo` shell
[accepts it too since v4.0](https://docs.mongodb.com/v4.0/mongo/#mongodb-instance-on-a-remote-host). Using
a v4.0+ mongo shell is a recommended way to debug connection URI validity from
the command line.

!!! admonition ""

    Since Percona Backup for MongoDB must authenticate in MongoDB, we recommend specifying the authentication database associated with the `pbm` user’s credentials in the connection URI string using the `authSource` option.

    The [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) specification also allows specifying the authentication database via the `defaultauthdb` component. However, in this case, Percona Backup for MongoDB makes a backup of only this specified database.

    If both `authSource` and `defaultauthdb` are unspecified, the authentication database defaults to the `admin` database.

The [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) specification
includes several non-default options you may need to use. For example the TLS
certificates/keys needed to connect to a cluster or non-sharded replica set with
network encryption enabled are “tls=true” plus “tlsCAFile” and/or
“tlsCertificateKeyFile” (see [tls options](https://docs.mongodb.com/manual/reference/connection-string/#tls-options)).

### The `pbm-agent` connection string

**pbm-agent** processes should connect to their localhost `mongod` with a standalone type of connection.

### The `pbm` CLI connection string

The `pbm` CLI will ultimately connect to the replica set with the
PBM Control Collections.


* In a non-sharded replica set it is simply that replica set.
* In a cluster it is the config server replica set.

You do not necessarily have to provide that connection string. If you provide
a connection to any live node (shard, configsvr, or non-sharded replica set
member), `pbm` CLI will automatically determine the right hosts and establish a new connection to those instead.

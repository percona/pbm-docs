# Authentication

Percona Backup for MongoDB has no authentication and authorization subsystem of its own —— it uses that of MongoDB. This means that `pbm` CLI and `pbm-agent` require only a valid MongoDB connection URI string for the `pbm` user.

For the S3-compatible remote storage authentication config, see
[Percona Backup for MongoDB configuration in a cluster (or non-sharded replica set)](../reference/config.md).

## MongoDB connection strings

Percona Backup for MongoDB uses [MongoDB Connection URI :octicons-link-external-16:](https://docs.mongodb.com/manual/reference/connection-string/) strings to open
MongoDB connections. Neither `pbm` CLI nor `pbm-agent` accept legacy-style
command-line arguments for `--host`, `--port`, `--user`, `--password`,
etc. as the `mongo` shell or `mongodump` command does.


=== "The `pbm-agent` connection string"

     The `pbm-agent` processes should connect to their localhost `mongod` with a standalone type of connection.

     ```{.bash data-prompt="$"}
     pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     ```

     Alternatively:

     ```{.bash data-prompt="$"} 
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     pbm-agent
     ```
     
     Replace the `pbmuser:secretpwd` with the credentials of [the user who owns the pbm process](../install/initial-setup.md#create-the-pbm-user).

=== "The `pbm` CLI connection string"

     ```{.bash data-prompt="$"}
     pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     ```

     Alternatively:

     ```{.bash data-prompt="$"}
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     $ pbm status
     ```
     
     Replace the `pbmuser:secretpwd` with the credentials of [the user who owns the pbm process](../install/configure-authentication.md#create-the-pbm-user)

     The `pbm` CLI will ultimately connect to the replica set with PBM Control Collections.

       * In a non-sharded replica set it is simply that replica set.
       * In a cluster it is the config server replica set.

     You do not necessarily have to provide that connection string. If you provide a connection to any live node (shard, configsvr, or non-sharded replica set member), `pbm` CLI will automatically determine the right hosts and establish a new connection to those instead.

The connection URI above is the format that MongoDB drivers have accepted universally
since approximately the release time of MongoDB server v3.6. The `mongo` shell
[has accepted it too since v4.0](https://docs.mongodb.com/v4.0/mongo/#mongodb-instance-on-a-remote-host). Using
a v4.0+ mongo shell is a recommended way to debug connection URI validity from
the command line.

!!! admonition ""

    Since Percona Backup for MongoDB must authenticate in MongoDB, we recommend specifying the authentication database associated with the `pbm` user’s credentials in the connection URI string using the `authSource` option.

    The [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) specification also allows specifying the authentication database via the `defaultauthdb` component. However, in this case, Percona Backup for MongoDB makes a backup of only this specified database.

    If both `authSource` and `defaultauthdb` are unspecified, the authentication database defaults to the `admin` database.

The [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) specification
includes several non-default options you may need to use. For example, the TLS
certificates/keys needed to connect to a cluster or non-sharded replica set with
network encryption enabled are “tls=true” plus “tlsCAFile” and/or
“tlsCertificateKeyFile” (see [tls options](https://docs.mongodb.com/manual/reference/connection-string/#tls-options)).

### Read Concern / Write Concern configuration

!!! admonition "Version added: [2.5.0](../release-notes/2.5.0.md)"

By default, PBM requires the majority level of acknowledgement among replica set members for read and write operations in MongoDB. This level is controlled by `readConcern` and `writeConcern` settings. 

If your cluster loses majority or is configured to operate without it, you can decrease the level for `readConcern` and `writeConcern` so that PBM remains operational and can make backups.



Specify new values in MongoDB connection URI string as follows:

=== "The pbm-agent connection string"    

    ```
    pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    ```  

    Alternatively:    

    ```
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    pbm-agent
    ```    

=== "The `pbm` CLI connection string"    

    ```{.bash data-prompt="$"}
    pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    ```    

    Alternatively:    

    ```{.bash data-prompt="$"}
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    $ pbm status
    ```

Supported values are:

* For `readConcern` – `local`
* For `writeConcern` – the number of nodes to acknowledge writes. A zero value is not supported.

For correct functioning of PBM, we recommend to change values for both options.

To restore from a backup, first configure your cluster to have the majority. Then [make a restore](../usage/restore.md).  







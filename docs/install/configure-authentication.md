# Configure authentication in MongoDB

Percona Backup for MongoDB uses the authentication and authorization subsystem of MongoDB. This means that to authenticate Percona Backup for MongoDB, you need to:

* [Create a corresponding `pbm` user](#create-the-pbm-user) in the `admin` database for each replicaset
* [Set a valid MongoDB connection URI string for **pbm-agent**](#set-the-mongodb-connection-uri-for-pbm-agent) 
* [Set a valid MongoDB connection URI string for `pbm` CLI](#set-the-mongodb-connection-uri-for-pbm-cli)

## Create the `pbm` user

!!! info

    Each PBM agent connects to its local node. To configure authentication, create the required user and role **on the primary node** of each replica set. 

    - For a standalone replica set, perform these steps on its primary node.
    - For a sharded cluster, perform these steps on the primary node of every shard replica set, as well as on the config server replica set.
    
1. Create the role that allows any action on any resource.

     ```javascript
     db.getSiblingDB("admin").createRole({ "role": "pbmAnyAction",
           "privileges": [
              { "resource": { "anyResource": true },
                "actions": [ "anyAction" ]
              }
           ],
           "roles": []
        });
     ```

2. Create the user and assign the role you created to it.

     ```javascript
     db.getSiblingDB("admin").createUser({user: "pbmuser",
            "pwd": "secretpwd",
            "roles" : [
               { "db" : "admin", "role" : "readWrite", "collection": "" },
               { "db" : "admin", "role" : "backup" },
               { "db" : "admin", "role" : "clusterMonitor" },
               { "db" : "admin", "role" : "restore" },
               { "db" : "admin", "role" : "pbmAnyAction" }
            ]
         });
     ```

You can change the `username` and `password` values and specify other options for the `createUser` command as you require. But you must grant this user the roles as shown above.


!!! tip

    To view all the host+port lists for the shard replica sets in a cluster, run the following command:

    ```javascript
    db.getSiblingDB("config").shards.find({}, {host: true, _id: false})
    ```

## Set the MongoDB connection URI for `pbm-agent`

!!! info 
    
    Execute this step on each node where `pbm-agent` is installed.

The `pbm-agent.service` systemd unit file includes the location of the environment file. You set the MongoDB URI connection string for the `PBM_MONGODB_URI` variable within the environment file for every `pbm-agent`. 

??? tip "How to find the environment file"

    In Ubuntu and Debian, the pbm-agent.service systemd unit file is at the path `/lib/systemd/system/pbm-agent.service`.

    In Red Hat and CentOS, the path to this file is `/usr/lib/systemd/system/pbm-agent.service`.

    **Example of pbm-agent.service systemd unit file**

    ```init
    [Unit]
    Description=pbm-agent
    After=time-sync.target network.target

    [Service]
    EnvironmentFile=-/etc/default/pbm-agent
    Type=simple
    User=pbm
    Group=pbm
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent

    [Install]
    WantedBy=multi-user.target
    ```
   
=== ":material-debian: On Debian and Ubuntu"    

    Edit the environment file `/etc/default/pbm-agent`. Specify the MongoDB connection URI string using the credentials of the `pbm` user you created previously. Ensure that the pbm-agent connects only to its local mongod node by providing the URI in the following format:

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

    * Replace <pbmuser:secretpwd> with the actual credentials you assigned to the pbm user.
    * `localhost:27017` ensures the agent connects only to the local `mongod` instance.
    * `authSource=admin` points authentication to the `admin` database, where the pbm user was created.

=== ":material-redhat: On Red Hat Enterprise Linux and derivatives"

    Edit the environment file `/etc/sysconfig/pbm-agent` Specify the MongoDB connection URI string using the credentials of the `pbm` user you created previously. Ensure that the pbm-agent connects only to its local mongod node by providing the URI in the following format:

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

    * Replace <pbmuser:secretpwd> with the actual credentials you assigned to the pbm user.
    * `localhost:27017` ensures the agent connects only to the local `mongod` instance.
    * `authSource=admin` points authentication to the `admin` database, where the pbm user was created.

To improve the security of the file, you can change its owner to the user that is configured in systemd, and set the file permission so that only the owner and root can read from it. 

!!! important

    Each **pbm-agent** process needs to connect to its localhost `mongod` node with a standalone type of connection. Avoid using the replica set URI with **pbm-agent** as it can lead to an unexpected behavior.
    
    For sharded clusters, **pbm-agent** on each shard member also needs to be able to reach the config server replica set over the network. The agent auto-discovers the config server's URI by querying the `db.system.version` collection. 

    Note that the MongoDB connection URI for `pbm-agent` is different from the connection string required by pbm CLI.
    
### Passwords with special characters

If the password includes special characters like `#`, `@`, `/` and so on, you must convert these characters using the [percent-encoding mechanism](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) when passing them to Percona Backup for MongoDB. For example, the password `secret#pwd` should be passed as follows in `PBM_MONGODB_URI`:

```
PBM_MONGODB_URI="mongodb://pbmuser:secret%23pwd@localhost:27017/?authSource=admin"
```

## Set the MongoDB connection URI for `pbm CLI`

!!! info 
    
    Execute this step only on the hosts where you will run `pbm` CLI commands for backup or restore.

Set the MongoDB URI connection string for `pbm` CLI as an environment variable in your shell. This allows you to run `pbm` commands without specifying the `--mongodb-uri` flag each time.

=== "Replica set"
   
    In a non-sharded replica set, point PBM CLI to the replica set. The following command is the example how to define the MongoDB URI connection string for a replica set with the replica set members `mongors1:27017`, `mongors2:27017` and `mongors3:27017`:

    ```
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongors1:27017,mongors2:27017,mongors3:27017/?authSource=admin&replSetName=xxxx"
    ```
    
=== "Sharded clusters"

    In a sharded cluster, point PBM CLI to the config server replica set. The following command is the example how to define the MongoDB URI connection string for a sharded cluster with the config servers `mongocfg1:27017`, `mongocfg2:27017` and `mongocfg3:27017`:
    
    ```
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocfg1:27017,mongocfg2:27017,mongocfg3:27017/?authSource=admin&replSetName=xxxx"
    ```

!!! important 
   
    The pbm CLI needs to connect to the replica set that stores PBM Control Collections. The MongoDB connection URI is different from the one required by `pbm-agent`.
    
For more information about the connection string to specify, refer to the [pbm connection string](../details/authentication.md#mongodb-connection-strings) section.

If you are using external authentication methods in MongoDB, see [External authentication support in Percona Backup for MongoDB](../details/authentication.md#external-authentication-support-in-percona-backup-for-mongodb) section for configuration guidelines.


## Next steps

[Configure backup storage :material-arrow-right:](backup-storage.md){.md-button}

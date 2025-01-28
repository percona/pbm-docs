# Configure authentication in MongoDB

Percona Backup for MongoDB uses the authentication and authorization subsystem of MongoDB. This means that to authenticate Percona Backup for MongoDB, you need to:

* [Create a corresponding `pbm` user](#create-the-pbm-user) in the `admin` database 
* [Set a valid MongoDB connection URI string for **pbm-agent**](#set-the-mongodb-connection-uri-for-pbm-agent) 
* [Set a valid MongoDB connection URI string for `pbm` CLI](#set-the-mongodb-connection-uri-for-pbm-cli)

## Create the `pbm` user

!!! info

    Execute this step on a primary node of each replica set. In a sharded cluster, this means on every shard replica set and the config server replica set.
    
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

You can specify the `username` and `password` values and other options of the `createUser` command as you require so long as the roles shown above are granted.


!!! tip

    To list all the host+port lists for the shard replica sets in a cluster, run the following command:

    ```javascript
    db.getSiblingDB(“config”).shards.find({}, {“host”: true, “_id”: false}) 
    ```

    The replica set name at the *front* of these “host” strings will have to be placed as a “/?replicaSet=xxxx” argument in the parameters part of the connection URI (see below).

## Set the MongoDB connection URI for `pbm-agent`

!!! info
    
    Execute this step on each node where `pbm-agent` is installed.

!!! important

    Each **pbm-agent** process needs to connect to its localhost `mongod` node with a standalone type of connection. Avoid using the replica set URI with **pbm-agent** as it can lead to unexpected behaviour.
    
    For sharded clusters, **pbm-agent** on each shard member also need to be able to connect to the config server replica set. The agents auto-discover the config server URI by querying the `db.system.version` collection. 

    Note that the MongoDB connection URI for `pbm-agent` is different from the connection string required by pbm CLI.

The `pbm-agent.service` systemd unit file includes the location of the environment file. You set the MongoDB URI connection string for the  `PBM_MONGODB_URI` variable within the environment file for every **pbm-agent**.

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

    Edit the environment file `/etc/default/pbm-agent` and specify the MongoDB connection URI string for the `pbm` user to the local `mongod` node.

    For example, if `mongod` node listens on port 27017, the MongoDB connection URI string will be the following:

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

=== ":material-redhat: On Red Hat Enterprise Linux and derivatives"

    Edit the environment file `/etc/sysconfig/pbm-agent` and specify the MongoDB connection URI string for the `pbm` user to the local `mongod` node. 

    For example, if `mongod` node listens on port 27017, the MongoDB connection URI string will be the following:

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

### Passwords with special characters

If the password includes special characters like `#`, `@`, `/` and so on, you must convert these characters using the [percent-encoding mechanism](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) when passing them to Percona Backup for MongoDB. For example, the password `secret#pwd` should be passed as follows in `PBM_MONGODB_URI`:

```
PBM_MONGODB_URI="mongodb://pbmuser:secret%23pwd@localhost:27017/?authSource=admin"
```

## Set the MongoDB connection URI for `pbm CLI`

!!! info 
    
    Execute this step only on a host at which you will use `pbm` CLI.

!!! important 
   
    The pbm CLI needs to connect to the replica set that stores PBM Control Collections. The MongoDB connection URI format is different from the one required by `pbm-agent`.

    In a non-sharded replica set it is simply that replica set. In a sharded cluster it is the config server replica set.

Set the MongoDB URI connection string for `pbm` CLI in your shell. This allows you to call `pbm` commands without the `--mongodb-uri` flag.

Use the following command:

```
export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&replSetName=xxxx"
```

For more information about what connection string to specify, refer to the [pbm connection string](../details/authentication.md#mongodb-connection-strings) section.

If you are using external authentication methods in MongoDB, see [External authentication support in Percona Backup for MongoDB](../details/authentication.md#external-authentication-support-in-percona-backup-for-mongodb) section for configuration guidelines.


## Next steps

[Configure backup storage :material-arrow-right:](backup-storage.md){.md-button}

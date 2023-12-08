# Configure authentication in MongoDB

Percona Backup for MongoDB uses the authentication and authorization subsystem  of MongoDB. This means that to authenticate Percona Backup for MongoDB, you need to:

* [Create a corresponding `pbm` user](#create-the-pbm-user) in the `admin` database 
* [Set a valid MongoDB connection URI string for **pbm-agent**](#set-the-mongodb-connection-uri-for-pbm-agent) 
* [Set a valid MongoDB connection URI string for `pbm` CLI](#set-the-mongodb-connection-uri-for-pbm-cli)

## Create the `pbm` user

<i info>:material-information: Info:</i> Execute this step on a primary node of each replica set. In a sharded cluster, this means on every shard replica set and the config server replica set.
{.power-number}
    
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

<i info>:material-information: Info:</i> Execute this step needs on each node where `pbm-agent` is installed.

A **pbm-agent** process connects to its localhost `mongod` node with a standalone type of connection. 

To set the MongoDB URI connection string means to configure a service init script (`pbm-agent.service` systemd unit file) that runs a **pbm-agent**.

The `pbm-agent.service` systemd unit file includes the environment file. You set the MongoDB URI connection string for the  `PBM_MONGODB_URI` variable within the environment file for every **pbm-agent**.

??? tip "How to find the environment file"

    The path to the environment file is specified in the `pbm-agent.service` systemd unit file.

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

=== "On Debian and Ubuntu Linux"

    Edit the environment file `/etc/default/pbm-agent` and specify the MongoDB connection URI string for the `pbm` user to the local `mongod` node.

    For example, if `mongod` node listens on port 27017, the MongoDB connection URI string will be the following:

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

=== "On Red Hat Enterprise Linux and derivatives"

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

<i info>:material-information: Info:</i> Execute this step only on a host at which you will use `pbm` CLI.

Set the MongoDB URI connection string for `pbm` CLI in your shell. This allows you to call `pbm` commands without the `--mongodb-uri` flag.

Use the following command:

```
export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&replSetName=xxxx"
```

For more information about what connection string to specify, refer to the [pbm connection string](../details/authentication.md#mongodb-connection-strings-a-reminder-or-primer) section.

## External authentication support in Percona Backup for MongoDB

In addition to SCRAM, Percona Backup for MongoDB supports other [authentication methods](https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html) that you use in MongoDB or Percona Server for MongoDB.

For external authentication, you create the `pbm` user in the format used by the authentication system and set the MongoDB connection URI string to include both the authentication method and authentication source.

For example, for [Kerberos authentication](https://docs.percona.com/percona-server-for-mongodb/6.0/authentication.html#kerberos-authentication), create the `pbm` user in the `$external` database in the format `<username@KERBEROS_REALM>` (e.g. [pbm@PERCONATEST.COM](mailto:pbm@PERCONATEST.COM)).

Specify the following string for MongoDB connection URI:

```
PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"
```

Note that you must first obtain the ticket for the `pbm` user with the `kinit` command before you start the **pbm-agent**:

```{.bash data-prompt="$"}
$ sudo -u {USER} kinit pbm
```

Note that the `{USER}` is the user that you will run the `pbm-agent` process.

For [authentication and authorization via Native LDAP](https://docs.percona.com/percona-server-for-mongodb/6.0/authorization.html#authentication-and-authorization-with-direct-binding-to-ldap), you only create roles for LDAP groups in MongoDB as the users are stored and managed on the LDAP server. However, you still define the `$external` database as your authentication source:

```
PBM_MONGODB_URI="mongodb://<user>:<password>@<hostname>:27017/?authMechanism=PLAIN&authSource=%24external&replSetName=xxxx"
```

When using [AWS IAM authentication](), create the `pbm` user in the `$external` database with the username that contains the ARN of the IAM user/role.


=== "User authentication"

     ```
     arn:aws:iam::<ARN>:user/<user_name>
     ```

=== "Role authentication"

     ```
     arn:aws:iam::<ARN>:role/<role_name>
     ```

The MongoDB connection URI string then looks like the following:

```
PBM_MONGODB_URI="mongodb://<aws_access_key_id>:<aws_secret_access_key>@<hostname>:27017/?authMechanism=MONGODB-AWS&authSource=%24external&replSetName=xxxx"
```

## Next steps

[Configure backup storage :material-arrow-right:](backup-storage.md){.md-button}
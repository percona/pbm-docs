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

     ```bash
     pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     ```

     Alternatively:

     ```bash 
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     pbm-agent
     ```
     
     Replace the `pbmuser:secretpwd` with the credentials of [the user who owns the pbm process](../install/configure-authentication.md#create-the-pbm-user).

=== "The `pbm` CLI connection string"

     ```bash
     pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     ```

     Alternatively:

     ```bash
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     pbm status
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

    ```bash
    pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    ```  

    Alternatively:    

    ```bash
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    pbm-agent
    ```    

=== "The `pbm` CLI connection string"    

    ```bash
    pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    ```    

    Alternatively:    

    ```bash
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    pbm status
    ```

Supported values are:

* For `readConcern` – `local`
* For `writeConcern` – the number of nodes to acknowledge writes. A zero value is not supported.

For correct functioning of PBM, we recommend to change values for both options.

To restore from a backup, first configure your cluster to have the majority. Then [make a restore](../usage/restore.md).  

## External authentication support in Percona Backup for MongoDB

In addition to SCRAM, Percona Backup for MongoDB supports other [authentication methods :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html) that you use in MongoDB or Percona Server for MongoDB.

For external authentication, you create the `pbm` user in the format used by the authentication system and set the MongoDB connection URI string to include both the authentication method and authentication source.

### Kerberos

For [Kerberos authentication :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html#kerberos-authentication), create the `pbm` user in the `$external` database in the format `<username@KERBEROS_REALM>` (e.g. [pbm@PERCONATEST.COM](mailto:pbm@PERCONATEST.COM)).

You can choose any of these methods to authenticate `pbm` user against Kerberos:

=== "Using a Keytab (Recommended)"

     1. Set the env variable `KRB5_CLIENT_KTNAME` with the path to the generated keytab for `pbm` user. This way no password is required to get the ticket. 
     
         ```bash
         export KRB5_CLIENT_KTNAME=/path/to/keytab
         ```
     
     2. Obtain the ticket for the `pbm` user with the `kinit` command before you start the **pbm-agent**:
     
         ```bash
         sudo -u {USER} kinit -t /path/to/keytab pbm@PERCONATEST.COM
         ```
     
         Note that the `{USER}` is the user that you will run the `pbm-agent` process. PBM doesn't refresh its ticket, so when it expires you need to get a new one.
     
     3. Specify the following string for MongoDB connection URI with only the username:
     
         ```bash
         PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"
         ```

=== "Requesting a ticket manually"

     1. Obtain the ticket for the `pbm` user with the `kinit` command before you start the **pbm-agent**. Kerberos will prompt you for the password and issue a Ticket-Granting Ticket (TGT):
     
         ```bash
         sudo -u {USER} kinit pbm@PERCONATEST.COM
         ```
     
         Note that the `{USER}` is the user that you will run the `pbm-agent` process. PBM doesn't refresh its ticket, so when it expires you need to get a new one.   
     
     2. Specify the following string for MongoDB connection URI with only the username:
     
         ```bash
         PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"
         ```

=== "Using username and password"

     You can authenticate using a connection string URI specifying your URL-encoded Kerberos principal, password, and the address of your MongoDB server:
     
     ```bash
     PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>:<PASSWORD>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"
     ```

### LDAP binding

For [authentication and authorization via Native LDAP :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authorization.html#authentication-and-authorization-with-direct-binding-to-ldap), you only create roles for LDAP groups in MongoDB as the users are stored and managed on the LDAP server. However, you still define the `$external` database as your authentication source:

```bash
PBM_MONGODB_URI="mongodb://<user>:<password>@<hostname>:27017/?authMechanism=PLAIN&authSource=%24external&replSetName=xxxx"
```

### AWS IAM

When using [AWS IAM authentication :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/aws-iam.html), create the `pbm` user in the `$external` database with the username that contains the ARN of the IAM user/role.


=== ":fontawesome-regular-user: User authentication"

     ```
     arn:aws:iam::<ARN>:user/<user_name>
     ```

=== ":material-cloud-key-outline: Role authentication"

     ```
     arn:aws:iam::<ARN>:role/<role_name>
     ```

The MongoDB connection URI string then looks like the following:

```bash
PBM_MONGODB_URI="mongodb://<aws_access_key_id>:<aws_secret_access_key>@<hostname>:27017/?authMechanism=MONGODB-AWS&authSource=%24external&replSetName=xxxx"
```

### AWS EKS

If Percona Backup for MongoDB runs in Amazon Elastic Kubernetes Service (EKS) (e.g. as Percona Operator for MongoDB), it accesses the AWS S3 storage and other services using the credentials stored in the IAM role associated with the service account in EKS and assigned to the Pod where Percona Backup for MongoDB is deployed.  

This saves you from creating and passing the AWS credentials to Pods explicitly thus increasing the overall security of your deployment.

To learn more about managing access to EKS, see [Learn how EKS Pod Identity grants pods access to AWS services :octicons-link-external-16:](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html).

For how to configure Percona Operator for MongoDB to use AWS S3 storage, refer to the [Configure storage for backups :octicons-link-external-16:](https://docs.percona.com/percona-operator-for-mongodb/backups-storage.html#amazon-s3-or-s3-compatible-storage) documentation.








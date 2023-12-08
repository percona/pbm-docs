# Run Percona Backup for MongoDB in a Docker container

Docker images of Percona Backup for MongoDB are hosted publicly on [Docker Hub](https://hub.docker.com/repository/docker/percona/percona-backup-mongodb).

For more information about using Docker, see the [Docker Docs](https://docs.docker.com/).

!!! note

    Make sure that you are using the latest version of Docker. The ones provided via apt and yum may be outdated and cause errors.

    By default, Docker will pull the image from Docker Hub if it is not available locally.

## Prerequisites

* You need to deploy MongoDB or Percona Server for MongoDB. See [what MongoDB deployments are supported](../details/deployments.md).
* [Create the pbm user](initial-setup.md#create-the-pbm-user) in your deployment. You will need this user credentials to start Percona Backup for MongoDB container. 

## Start Percona Backup for MongoDB 

Start Percona Backup for MongoDB container with the following command:


```{.bash data-prompt="$"}
$ docker run --name <container-name> -e PBM_MONGODB_URI="mongodb://<PBM_USER>:<PBM_USER_PASSWORD>@<HOST>:<PORT>" -d percona/percona-backup-mongodb:<tag>-multi
```

Where:

* `container-name` is the name you want to assign to your container.
* `PBM_MONGODB-URI` is a [MongoDB Connection URI](https://docs.mongodb.com/manual/reference/connection-string/) string used to connect to MongoDB nodes. See the [Initial setup](initial-setup.md) how to create the PBM user. 
* `tag-multi` is the tag specifying the version you need. For example, `{{release}}-multi`. The `multi` part of the tag serves to identify the architecture (x86_64 or ARM64) and pull the respective image. See the [full list of tags](https://hub.docker.com/r/perconalab/percona-backup-mongodb/tags).

Note that every MongoDB node (including replica set secondary members and config server replica set nodes) requires a separate instance of Percona Backup for MongoDB. Thus, a typical, 3-node MongoDB replica set requires three instances of Percona Backup for MongoDB.

## Set up Percona Backup for MongoDB 

Percona Backup for MongoDB requires the remote storage where to store data. Use the following commands to configure it:

1. Start a Bash session:
	
    ```{.bash data-prompt="$"}
    $ docker exec -it --name <container-name> bash
    ```

2. Create a YAML configuration file:

	```{.bash data-prompt="$"}
	$ vi /tmp/pbm_config.yaml
	```
	
3. Specify remote storage parameters in the config file. The following example is for S3-compatible backup storage. Check what [other storages are supported](../details/storage-configuration.md) and [examples of storage configurations](../details/storage-config-example.md):

	```yaml
	storage:
		type: s3
		s3:
		  region: <your-region-here>
		  bucket: <your-bucket-here>
	      credentials:
	        access-key-id: <your-access-key-id-here>
		secret-access-key: <your-secret-key-here>
	```

4. Upload the config file: 
	
	```{.bash data-prompt="$"}
	$ pbm config --file /tmp/pbm_config.yaml
	```

	The command output displays your uploaded configuration.

## Run Percona Backup for MongoDB

Percona Backup for MongoDB command line utility (`pbm`) provides the set of commands to control backups: create, restore, cancel backups, etc. 

For example, to start a backup, use the following command:

```{.bash data-prompt="$"}
$ docker exec -it --name <container-name> pbm backup
```

where `<container-name>` is the name you assigned to the container and `pbm backup` is the command to start a backup.

In the same way you can run other pbm commands. Find the full list of available commands in [Percona Backup for MongoDB reference](https://docs.percona.com/percona-backup-mongodb/reference/pbm-commands.html).

## Next steps

[List backups](../usage/list-backup.md){.md-button}
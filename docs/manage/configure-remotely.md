# Configure Percona Backup for MongoDB remotely

!!! admonition "Version added: 2.0.1"

To apply or update the configuration, Percona Backup for MongoDB reads the configuration file on the filesystem. When running Percona Backup for MongoDB remotely (in a cloud as Docker containers or pods in Kubernetes), you must upload the configuration file to the remote host's filesystem.  

Starting with version 2.0.1, you can configure Percona Backup for MongoDB remotely. You manage the configuration file locally and use the pipeline to pass the file's contents to Percona Backup for MongoDB on a remote host/running in a container. As a result, your DBAs spend less time on administering Percona Backup for MongoDB and can focus on other activities instead.

Here’s how to configure Percona Backup for MongoDB remotely:

1. Create/update the configuration file (for example, `/etc/pbm_config.yaml`)
2. Create an environment variable for the path to the configuration file

    ```sh
    export CONFIG_PATH="/etc/pbm_config.yaml"
    ```

3. Pass the configuration file contents to Percona Backup for MongoDB. For example, if you run Percona Backup for MongoDB in Docker, use one of the following commands:
   
    * Connect to the existing container and pass the configuration:

        ```sh
        cat "$CONFIG_PATH" | docker compose exec -T $SERVICE_NAME pbm config --file="-"
        ```

        Replace the `$SERVICE_NAME` with your [service name](https://docs.docker.com/compose/compose-file/#services-top-level-element).

    * Create a new container to pass the configuration and exit: 

        ```sh
        cat "$CONFIG_PATH" | docker run -i --env PBM_MONGODB_URI="mongodb://<PBM_USER>:<PBM_USER_PASSWORD>@<HOST>:<PORT>" --network=$NET_ID $CONTAINER_ID pbm config --file="-"
        ```

        Specify the valid PBM_MONGODB_URI connection string, the ID of the network the container will connect to and the container ID.




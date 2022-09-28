# Configure Percona Backup for MongoDB remotely

!!! admonition "Version added: 2.0.1"

To apply or update the configuration, Percona Backup for MongoDB reads the configuration file on the filesystem. When running Percona Backup for MongoDB remotely (in a cloud as Docker containers or pods in Kubernetes), you must upload the configuration file to the remote host's filesystem.  

Starting with version 2.0.1, you can configure Percona Backup for MongoDB remotely. You manage the configuration file locally and use the pipeline to pass the file's contents to Percona Backup for MongoDB on a remote host/running in a container. As a result, your DBAs spend less time on administering Percona Backup for MongoDB and can focus on other activities instead.

Here’s remote configuration works:

1. Create/update the configuration file
2. Create an environment variable for the path to the configuration file

    ```sh
    export $CONFIG_PATH="/etc/pbm_config.yaml"
    ```

3. Pass the configuration file contents to Percona Backup for MongoDB. For example, if you run PBM in Docker, use a separate job:

    ```sh
    cat "$CONFIG_PATH" | docker run $IMAGE_ID pbm config --file="-"
    ```

As a result, the command prints the file contents and then Docker runs a separate image that passes the file contents to PBM and exits.   
 

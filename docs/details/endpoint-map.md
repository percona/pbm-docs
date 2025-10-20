# Multiple endpoints to the same storage

!!! admonition "Version added: [2.8.0](../release-notes/2.8.0.md)" 

In environments where `pbm-agents` run on servers that are distributed across several data centers, accessing the same remote backup storage can become challenging. This can be due to complex network configurations or strict policies that prevent direct connections to the outside world. As a result, `pbm-agents` can't use the same URL to reach the storage, which is necessary for Percona Backup for MongoDB to work properly.

To address these challenges, you can configure custom endpoint URLs for specific nodes in the PBM configuration. This allows all `pbm-agents` to access the same storage while respecting the network settings of their data centers.

The supported storage types are: 

* AWS S3 
* MinIO and S3-compatible storage services 
* Microsoft Azure Blob storage 

Here's the example of the configuration file with the endpoint map:

=== ":fontawesome-brands-amazon: AWS S3"

    ```yaml
    storage:
        type: s3
        s3:
          endpointUrl: http://S3:9000
          endpointUrlMap:
            "node01:27017": "did.socf.s3.com"
            "node03:27017": "https://example.aws.s3.com"
          ...
    ```

=== ":simple-minio: MinIO and S3-compatible storage"

    ```yaml
    storage:
        type: minio
        minio:
          endpoint: localhost:9100
          endpointMap:
            "node01:27017": "did.socf.s3.com"
            "node03:27017": "https://example.min.io"
          ...
    ```

=== ":material-microsoft-azure: Microsoft Azure Blob storage"

    ```yaml
    storage:
        type: azure
        azure:
          endpointUrl: https://myaccount.blob.core.windows.net
          endpointUrlMap:
            "node01:27017": "did.socf.blob.core.windows.net"
            "node03:27017": "https://example.azure.blob.core.windows.net"
          ...
    ```


You can define the specific nodes for the `endpointUrlMap` parameter for AWS S3 and Azure or for the `endpointMap` for MinIO and S3-compatible storage. Not listed nodes use the endpoint defined for the `endpointUrl` / `endpoint` parameter. 

For the solution to work, you should also have the mapping mechanism in place. This mechanism should be able to map the custom endpoints to the main endpoint URL of the storage, routing the requests from `pbm-agents` to the storage and back seamlessly.

With this ability to control the endpoints for `pbm-agents` to reach the same storage, you reduce the administrative overhead on PBM configuration and ensure its proper functioning. 
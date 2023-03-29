# Automate access to S3 buckets for Percona Backup for MongoDB

When you run MongoDB and Percona Backup for MongoDB using AWS resources (on EC2 instances or using EKS), you can automate access to AWS S3 buckets for Percona Backup for MongoDB. Percona Backup for MongoDB uses the AWS environment variables and metadata to access S3 buckets so that you don’t have to explicitly specify the S3 credentials in the PBM configuration file. Thereby you control the access to your cloud infrastructure from a single place.

## IAM instance profile 

!!! admonition "Version added: 1.6.0"

IAM (Identity Access Management) is the AWS service that allows you to securely control access to AWS resources.

Using the IAM instance profile, you can automate access to S3 buckets for Percona Backup for MongoDB running on EC2 instance. The steps are the following:

1. Create the [IAM instance profile](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) and the permission policy within where you specify the access level that grants the access to S3 buckets.

2. Attach the IAM profile to an EC2 instance.

3. Configure an S3 storage bucket and verify the connection from the EC2 instance to it.


4. Provide the [remote storage information for PBM in a config file](../install/initial-setup.md#configure-remote-backup-storage). Leave the `s3.credentials` array empty
    
    ```yaml
    storage:
      type: s3
      s3:
       region: <your-S3-region>
       bucket: <bucket-name>
    ```

    !!! note

        If you specify S3 credentials, they override the EC2 instance environment variables and metadata, and are used for authentication instead.


5. Start the `pbm-agent` process

!!! admonition "See also"

    AWS documentation: [How can I grant my Amazon EC2 instance access to an Amazon S3 bucket?](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-access-s3-bucket/)

## IAM Roles for Service Accounts (IRSA)

!!! admonition "Version added: 2.0.3"

[IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) is the native way for AWS EKS (Amazon Elastic Kubernetes Service) to allow applications running in EKS pods to access the AWS API using permissions configured in AWS IAM roles.

To benefit from using the AWS IRSA credentials with PBM, the high-level steps are the following:

1. [Create a cluster](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eks-cluster.html) with `eksctl` and OIDC provider setup enabled. This feature works with EKS clusters version 1.13 and above.
2. Create an IAM role and specify the policy that defines the access to an S3 bucket.
3. Create a service account and annotate it with the IAM role.
3. Configure your pod by using the service account created in the previous step and assume the IAM role.
4. Provide the remote storage information for PBM in a config file. Leave the `s3.credentials` array empty, since PBM uses the `AWS_ROLE_ARN`/`AWS_WEB_IDENTITY_TOKEN_FILE` environment variables which are either automatically provided (i.e. injected by Kubernetes mutating admission controller in EKS) or which you can define manually (if you don't want to the admission controller to modify your pods)


!!! note 

    If IRSA-related credentials are defined, they have the priority over any IAM instance profile. However, if you intentionally specify S3 credentials in PBM configuration file, they override any IRSA/IAM instance profile related credentials and are used for authentication instead.

!!! admonition "See also"

    AWS documentation: 

    * [Introducing fine-grained IAM roles for service accounts](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
    * [How do I use the IAM roles for service accounts (IRSA) feature with Amazon EKS to restrict access to an Amazon S3 bucket?](https://aws.amazon.com/premiumsupport/knowledge-center/eks-restrict-s3-bucket/)



*[EC2]: Elastic Compute Cloud
*[EKS]: Elastic Kubernetes Service
*[IAM]: Identity Access Management
*[IRSA]: IAM Roles for Service Accounts
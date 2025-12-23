# Automate access to S3 buckets for Percona Backup for MongoDB

When you run Percona Backup for MongoDB in AWS environment (on EC2 instances or using EKS), you can automate access to AWS S3 buckets for Percona Backup for MongoDB. Percona Backup for MongoDB uses the environment variables and metadata to access S3 buckets so that you don’t have to explicitly specify the S3 credentials in the PBM configuration file. Thereby you control the access to your cloud infrastructure from a single place.

## Assume a role from an EC2 instance

You can configure Percona Backup for MongoDB to assume an IAM role. To make this work, the `pbm-agent` leverages the AWS environment variables to assume the specified role. The steps are the following:

1.  Ensure that the EC2 instance where `pbm-agent` is running has an [IAM instance profile :octicons-link-external-16:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) attached. The role associated with this instance profile must have a permissions policy that allows it to assume the target role.

    For example, if your target role is `arn:aws:iam::TARGET_ACCOUNT_ID:role/pbm-target-role`, create a policy with the following content and attach it to your EC2 instance role:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "sts:AssumeRole",
                "Resource": "arn:aws:iam::TARGET_ACCOUNT_ID:role/pbm-target-role"
            }
        ]
    }
    ```
    > Remember to replace `TARGET_ACCOUNT_ID` and `pbm-target-role` with your actual target account ID and role name.

2.  Create the target IAM role that PBM will assume. This involves setting up a trust relationship and attaching the necessary S3 permissions.

    *   **Trust Policy**: The trust policy of the target role must allow the EC2 instance's role to assume it.

        For example, if your EC2 instance role is `arn:aws:iam::EC2_ACCOUNT_ID:role/pbm-ec2-instance-role`, use the following trust policy for your target role:

        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::EC2_ACCOUNT_ID:role/pbm-ec2-instance-role"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
        ```
        > Remember to replace `EC2_ACCOUNT_ID` and `pbm-ec2-instance-role` with the account ID and role name of your EC2 instance.

    *   **Permissions Policy**: The target role must have a permissions policy attached that grants the necessary S3 access for PBM.

        Here is an example policy that grants the required permissions. Attach it to your target role:

        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:GetObject",
                        "s3:PutObject",
                        "s3:DeleteObject",
                        "s3:GetBucketLocation"
                    ],
                    "Resource": "arn:aws:s3:::your-pbm-bucket/*"
                },
                {
                    "Effect": "Allow",
                    "Action": "s3:ListBucket",
                    "Resource": "arn:aws:s3:::your-pbm-bucket"
                }
            ]
        }
        ```
        > Remember to replace `your-pbm-bucket` with the name of your S3 bucket.

3.  Set the `AWS_ROLE_ARN` environment variable to the ARN of the target role in the environment where you start the `pbm-agent` process. You can also set `AWS_SESSION_NAME` (optional, but recommended for audit trails).

    ```bash
    export AWS_ROLE_ARN=arn:aws:iam::ACCOUNT_ID:role/pbm-target-role
    export AWS_SESSION_NAME=pbm-session-$(hostname)
    ```

4.  In the [PBM configuration](../install/backup-storage.md) provide remote storage information, but leave the `s3.credentials` section empty. PBM will use the assumed role's credentials.

    ```yaml
    storage:
      type: s3
      s3:
       region: <your-S3-region>
       bucket: <bucket-name>
    ```

!!! note

    When `AWS_ROLE_ARN` is set, the credentials from the assumed role take precedence over the EC2 instance profile's credentials. However, if you specify `s3.credentials` in the PBM configuration file, they will override any other IAM-based credentials.


5. Restart the `pbm-agent` process.

!!! admonition "See also"

    AWS documentation: [How can I grant my Amazon EC2 instance access to an Amazon S3 bucket? :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-access-s3-bucket/)
    
## IAM Roles for Service Accounts (IRSA)

!!! admonition "Version added: 2.0.3"

[IRSA :octicons-link-external-16:](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) is the native way for AWS EKS (Amazon Elastic Kubernetes Service) to allow applications running in pods to access the AWS API, using permissions configured in AWS IAM roles. This is the recommended approach for granting S3 access to Percona Backup for MongoDB in an EKS environment.

To configure IRSA for PBM, follow these steps:

1. Set up an OIDC provider for your EKS cluster

IRSA requires an OpenID Connect (OIDC) provider to be associated with your cluster.

First, check if you already have an OIDC provider. Replace `<cluster_name>` with your EKS cluster's name.
```bash
aws eks describe-cluster --name <cluster_name> --query "cluster.identity.oidc.issuer" --output text
```
If the command returns a URL, your OIDC provider is already set up. If not, create one using `eksctl`. Replace `<region>` and `<cluster-name>` with your cluster's region and name.
```bash
eksctl utils associate-iam-oidc-provider --region <region> --cluster <cluster-name> --approve
```

2. Create an IAM policy for S3 access

This policy defines the permissions that PBM needs to access your S3 bucket.

Create a JSON file (e.g., `pbm-s3-policy.json`) with the following content. Remember to replace `your-pbm-bucket` with your actual bucket name.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::your-pbm-bucket/*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::your-pbm-bucket"
        }
    ]
}
```

Now, create the IAM policy using the AWS CLI.
```bash
aws iam create-policy --policy-name pbm-s3-access-policy --policy-document file://pbm-s3-policy.json
```

3. Create an IAM role

This role will be assumed by the Kubernetes service account used by your PBM pods.

First, create a trust policy JSON file (e.g., `pbm-trust-policy.json`). This policy allows your Kubernetes service account to assume the role. Replace `<account-id>`, `<region>`, and `<oidc-id>` with your AWS account ID, EKS cluster region, and the OIDC ID from step 1.

> **Note:** The OIDC ID is the unique identifier at the end of the OIDC issuer URL returned in step 1. For example, if the issuer URL is `https://oidc.eks.us-west-2.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE`, then the OIDC ID is `EXAMPLED539D4633E53DE1B71EXAMPLE`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account-id>:oidc-provider/oidc.eks.<region>.amazonaws.com/id/<oidc-id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<oidc-id>:aud": "sts.amazonaws.com",
          "oidc.eks.<region>.amazonaws.com/id/<oidc-id>:sub": "system:serviceaccount:<namespace>:<service-account-name>"
        }
      }
    }
  ]
}
```
> **Note:** Replace `<namespace>` and `<service-account-name>` with the namespace and name of the service account your PBM pods will use.

Create the IAM role:
```bash
aws iam create-role --role-name pbm-s3-access-role --assume-role-policy-document file://pbm-trust-policy.json
```

4. Attach the policy to the role

Attach the S3 access policy you created in step 2 to the role you created in step 3. Replace `<account-id>` with your AWS account ID.
```bash
aws iam attach-role-policy --role-name pbm-s3-access-role --policy-arn arn:aws:iam::<account-id>:policy/pbm-s3-access-policy
```

5. Annotate the Kubernetes service account

Now, you need to associate the IAM role with the Kubernetes service account that your `pbm-agent` pods use. This is done by annotating the service account.

First, get the ARN of the role you created:
```bash
aws iam get-role --role-name pbm-s3-access-role --query "Role.Arn" --output text
```

Then, annotate the service account. Replace `<namespace>`, `<service-account-name>`, and `<role_arn>` with the correct values.
```bash
kubectl annotate serviceaccount <service-account-name> -n <namespace> \
    eks.amazonaws.com/role-arn="<role_arn>"
```
If your pods are already running, you will need to restart them to apply the changes and inject the AWS environment variables.

6. Configure PBM storage

Finally, configure your PBM storage to use S3, but do not provide any credentials. PBM will automatically use the credentials provided by IRSA.

Your storage configuration should look like this:
```yaml
storage:
  type: s3
  s3:
   region: <your-s3-region>
   bucket: <your-pbm-bucket>
```

!!! note 
    If IRSA-related credentials are defined, they have priority over any IAM instance profile. However, if you intentionally specify S3 credentials in the PBM configuration file, they override any IRSA/IAM instance profile related credentials and are used for authentication instead.

!!! admonition "See also"

    AWS documentation: 

    * [Introducing fine-grained IAM roles for service accounts :octicons-link-external-16:](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
    * [How do I use the IAM roles for service accounts (IRSA) feature with Amazon EKS to restrict access to an Amazon S3 bucket? :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/eks-restrict-s3-bucket/)

*[EC2]: Elastic Compute Cloud
*[EKS]: Elastic Kubernetes Service
*[IAM]: Identity Access Management
*[IRSA]: IAM Roles for Service Accounts
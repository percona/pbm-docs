# Automate access to S3 buckets for Percona Backup for MongoDB

When you run Percona Backup for MongoDB in an AWS environment, you can automate access to S3 buckets without storing credentials in the PBM configuration file. PBM uses AWS environment variables and instance metadata to authenticate, so you control access to your cloud infrastructure from a single place.

Choose the method that matches your deployment:

- **EC2 instances** — configure `pbm-agent` to assume an IAM role using the instance profile attached to the EC2 instance.
- **EKS (Kubernetes)** — use [IAM Roles for Service Accounts (IRSA)](#iam-roles-for-service-accounts-irsa), the recommended approach for pod-level AWS authentication on EKS.

## Assume a role from an EC2 instance

When `pbm-agent` runs on an EC2 instance, it can assume a target IAM role using the instance profile credentials. Set the `AWS_ROLE_ARN` environment variable to tell PBM which role to assume.

### Prerequisites

- AWS CLI installed and configured with permissions to create and attach IAM policies and roles
- An EC2 instance with an [IAM instance profile :octicons-link-external-16:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) attached
- The name and account ID of the EC2 instance role associated with that profile
- An S3 bucket for PBM backup storage

### Steps

1. Ensure that the EC2 instance where `pbm-agent` is running has an [IAM instance profile :octicons-link-external-16:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) attached. The role associated with this instance profile must have a permissions policy that allows it to assume the target role.

    Create a file named `pbm-assume-role-policy.json` with the following content, replacing `TARGET_ACCOUNT_ID` and `pbm-target-role` with your target account ID and role name:

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

    Create the IAM policy and attach it to the EC2 instance role, replacing `EC2_ACCOUNT_ID` and `pbm-ec2-instance-role` with your values:

    ```bash
    aws iam create-policy \
        --policy-name pbm-assume-role-policy \
        --policy-document file://pbm-assume-role-policy.json

    aws iam attach-role-policy \
        --role-name pbm-ec2-instance-role \
        --policy-arn arn:aws:iam::EC2_ACCOUNT_ID:policy/pbm-assume-role-policy
    ```

2. Create the target IAM role that PBM will assume. This involves setting up a trust relationship and attaching the necessary S3 permissions.

    **Trust policy** — allows the EC2 instance role to assume the target role.

    Create a file named `pbm-target-trust-policy.json` with the following content, replacing `EC2_ACCOUNT_ID` and `pbm-ec2-instance-role` with the account ID and role name of your EC2 instance:

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

    Create the target IAM role:

    ```bash
    aws iam create-role \
        --role-name pbm-target-role \
        --assume-role-policy-document file://pbm-target-trust-policy.json
    ```

    **Permissions policy** — grants the target role the S3 access that PBM requires.

    Create a file named `pbm-s3-policy.json` with the following content, replacing `your-pbm-bucket` with your S3 bucket name:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject",
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::your-pbm-bucket/*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket",
                    "s3:GetBucketLocation"
                ],
                "Resource": "arn:aws:s3:::your-pbm-bucket"
            }
        ]
    }
    ```

    Create the IAM policy and attach it to the target role, replacing `TARGET_ACCOUNT_ID` with your target account ID:

    ```bash
    aws iam create-policy \
        --policy-name pbm-s3-access-policy \
        --policy-document file://pbm-s3-policy.json

    aws iam attach-role-policy \
        --role-name pbm-target-role \
        --policy-arn arn:aws:iam::TARGET_ACCOUNT_ID:policy/pbm-s3-access-policy
    ```

3. Set the `AWS_ROLE_ARN` environment variable to the ARN of the target role in the environment where you start the `pbm-agent` process. Setting `AWS_ROLE_SESSION_NAME` is optional but recommended for audit trails.

    ```bash
    export AWS_ROLE_ARN=arn:aws:iam::ACCOUNT_ID:role/pbm-target-role
    export AWS_ROLE_SESSION_NAME=pbm-session-$(hostname)
    ```

4. In the [PBM configuration](../install/backup-storage.md), provide the remote storage information but leave the `s3.credentials` section empty. PBM uses the assumed role’s credentials automatically.

    ```yaml
    storage:
      type: s3
      s3:
        region: <your-S3-region>
        bucket: <bucket-name>
    ```

5. Restart the `pbm-agent` process to apply the new environment variables.

!!! note

    When `AWS_ROLE_ARN` is set, the credentials from the assumed role take precedence over the EC2 instance profile’s credentials. If you explicitly specify `s3.credentials` in the PBM configuration file, those credentials override any IAM-based authentication.

!!! admonition "See also"

    AWS documentation: [How can I grant my Amazon EC2 instance access to an Amazon S3 bucket? :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-access-s3-bucket/)

## IAM Roles for Service Accounts (IRSA)

!!! admonition "Version added: 2.0.3"

[IRSA :octicons-link-external-16:](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) is the native AWS EKS mechanism for granting pods access to the AWS API using IAM roles. It is the recommended approach for granting S3 access to PBM in an EKS environment.

### Prerequisites

- AWS CLI and `kubectl` configured with access to your cluster
- `eksctl` installed (used to associate the OIDC provider)

### Steps

1. **Set up an OIDC provider for your EKS cluster.**

    IRSA requires an OpenID Connect (OIDC) provider associated with your cluster. Check whether one already exists:

    ```bash
    aws eks describe-cluster --name <cluster_name> --query "cluster.identity.oidc.issuer" --output text
    ```

    If the command returns a URL, the OIDC provider is already configured. If not, create one:

    ```bash
    eksctl utils associate-iam-oidc-provider --region <region> --cluster <cluster-name> --approve
    ```

    Note the OIDC issuer URL — you will need the ID at the end of the URL in subsequent steps. For example, if the URL is `https://oidc.eks.us-west-2.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE`, the OIDC ID is `EXAMPLED539D4633E53DE1B71EXAMPLE`.

2. **Create an IAM policy for S3 access.**

    This policy defines the permissions PBM needs to access your S3 bucket. Create a file named `pbm-s3-policy.json` with the following content, replacing `your-pbm-bucket` with your actual bucket name:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject",
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::your-pbm-bucket/*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket",
                    "s3:GetBucketLocation"
                ],
                "Resource": "arn:aws:s3:::your-pbm-bucket"
            }
        ]
    }
    ```

    Create the IAM policy:

    ```bash
    aws iam create-policy --policy-name pbm-s3-access-policy --policy-document file://pbm-s3-policy.json
    ```

3. **Create an IAM role with a trust policy.**

    This role will be assumed by the Kubernetes service account used by your PBM pods. Create a trust policy file named `pbm-trust-policy.json`, replacing `<account-id>`, `<region>`, `<oidc-id>`, `<namespace>`, and `<service-account-name>` with your values:

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

    Create the IAM role:

    ```bash
    aws iam create-role --role-name pbm-s3-access-role --assume-role-policy-document file://pbm-trust-policy.json
    ```

4. **Attach the S3 policy to the role.**

    Replace `<account-id>` with your AWS account ID:

    ```bash
    aws iam attach-role-policy --role-name pbm-s3-access-role --policy-arn arn:aws:iam::<account-id>:policy/pbm-s3-access-policy
    ```

5. **Annotate the Kubernetes service account.**

    Associate the IAM role with the Kubernetes service account that your `pbm-agent` pods use. First, retrieve the role ARN:

    ```bash
    aws iam get-role --role-name pbm-s3-access-role --query "Role.Arn" --output text
    ```

    Then annotate the service account, replacing `<namespace>`, `<service-account-name>`, and `<role-arn>` with the correct values:

    ```bash
    kubectl annotate serviceaccount <service-account-name> -n <namespace> \
        eks.amazonaws.com/role-arn="<role-arn>"
    ```

    If your pods are already running, restart them to apply the changes and inject the AWS environment variables.

6. **Configure PBM storage.**

    In the [PBM configuration](../install/backup-storage.md), specify the S3 storage without credentials. PBM automatically uses the credentials provided by IRSA:

    ```yaml
    storage:
      type: s3
      s3:
        region: <your-s3-region>
        bucket: <your-pbm-bucket>
    ```

!!! note

    IRSA credentials take priority over any IAM instance profile. If you explicitly specify `s3.credentials` in the PBM configuration file, those credentials override both IRSA and IAM instance profile authentication.

!!! admonition "See also"

    AWS documentation:

    * [Introducing fine-grained IAM roles for service accounts :octicons-link-external-16:](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
    * [How do I use the IAM roles for service accounts (IRSA) feature with Amazon EKS to restrict access to an Amazon S3 bucket? :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/eks-restrict-s3-bucket/)

*[EC2]: Elastic Compute Cloud
*[EKS]: Elastic Kubernetes Service
*[IAM]: Identity Access Management
*[IRSA]: IAM Roles for Service Accounts
# Complete Guide to Setup Nodejs App in EKS
## create a user with administrative permission or assign permissions required to manage:

* EKS clusters

* ECR (Amazon Elastic Container Registry)

* EC2 resources used by EKS

* IAM roles required for EKS

## After creation, we generate an access key and configure the AWS CLI on the local machine.

## 🧩 1. Create IAM User

1.Log in to the AWS Console

2.Go to IAM → Users

3.Click Create user

4.Enter username:
```bash
eks-demo
```

5.Leave Console access → Disabled (CLI only)

6.Click Create user

## 🛡️ 2. Attach Required Permissions

1.Select the user → Permissions → Add permissions → Attach policies directly.

Attach the following managed policies:
* AmazonEKSClusterPolicy
* AmazonEKSWorkerNodePolicy
* AmazonEKSServicePolicy
* AmazonEC2FullAccess
* IAMReadOnlyAccess
* AmazonEC2ContainerRegistryFullAccess
  
Or We can create adminstrative permisstion(optionaal):
* AdministratorAccess
* 
## 🔑 3. Create Access Key

1.Go to IAM → Users → eks-demo

2.Open Security credentials tab

3.Scroll to Access keys

4.Click Create access key

5.Select “Command Line Interface (CLI)”

6.Download .csv file containing:

* Access Key ID

* Secret Access Key

⚠️ This is the only time you can download the secret key.
Keep it safe.

.

## 🖥️ 4. Configure AWS CLI on Local Machine

Run:
```bash
aws configure
```
Enter:
```bash

AWS Access Key ID: <YOUR_KEY>
AWS Secret Access Key: <YOUR_SECRET>
Default region name: us-east-1   # or your region
Default output format: json
```

Verify configuration:
```bash

aws sts get-caller-identity
```

Expected output should show:

* "userId": "eks-demo",
* "account": "<your-account-id>"

# 📦 5. (Optional) Test ECR Login

Use the region where your ECR repo exists.
```bash
aws ecr get-login-password --region us-east-1 \
| docker login --username AWS \
--password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
```

If successful, Docker prints:
```bash
Login Succeeded
```


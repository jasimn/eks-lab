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


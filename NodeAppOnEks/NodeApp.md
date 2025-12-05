
# Complete Guide to Setup Nodejs App in EKS
---------------------
## Create a user with administrative permission or assign permissions required to manage:
* EKS clusters

* ECR (Amazon Elastic Container Registry)

* EC2 resources used by EKS

* IAM roles required for EKS

## After creation, we generate an access key and configure the AWS CLI on the local machine.
------------------------
##  🧩 step 1. Create IAM User

1.Log in to the AWS Console

2.Go to IAM → Users

3.Click Create user

4.Enter username:
```bash
eks-demo
```

5.Leave Console access → Disabled (CLI only)

6.Click Create user

## 🛡️ step 2. Attach Required Permissions

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
 ![App Screenshot](https://github.com/jasimn/eks-lab/blob/main/Screenshot%20from%202025-12-05%2015-23-39.png)

## 🔑  step 3. Create Access Key


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

--------------
## 🖥️ step 4. Configure AWS CLI on Local Machine
-------------------
## If aws-cli is not installed in your local machine please install aws-cli .
  
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

## 📦 step  5. (Optional) Test ECR Login

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
## step 6. — Create a Simple Node.js App
-------------------------

Create a folder and a file:
```bash
mkdir mynodeapp
cd mynodeapp
vim app.js
```
Add simple code:
```bash
const express = require("express");
const app = express();
app.get("/", (req, res) => {
  res.send("Hello from Node App on EKS!");
});
app.listen(3000, () => console.log("App running on port 3000"));
```

Initialize project:
```bash
npm init -y
npm install express
```
## json file should like this.
package.json:
```bash
{
  "name": "mynodeapp",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.2.1"
  }
}
```
-------------------------
# step  7 — Create Dockerfile
-------------------------

## Create Dockerfile:

vim Dockerfile


Put this inside:
```bash
FROM node:18

WORKDIR /app

# Copy only package.json first
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy rest of your app
COPY . .

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "app.js"]
```

## Build Docker image:
```bash
docker build -t mynodeapp:1.0 .
```

Test locally:
```bash
docker run -p 3000:3000 mynodeapp:1.0
```
-----------------------------
## step 8. — Push Image to AWS ECR (Elastic Container Registry)
-------------------------
3.1 Create ECR Repository
```bash
aws ecr create-repository --repository-name mynodeapp
```
 ![App Screenshot](https://github.com/jasimn/eks-lab/blob/main/Screenshot%20from%202025-12-05%2017-31-54.png)

3.2 Login to ECR
```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com
```
3.3 Tag & Push Image
```bash
docker tag mynodeapp:1.0 <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/mynodeapp:1.0
docker push <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/mynodeapp:1.0
```
![App Screenshot](https://github.com/jasimn/eks-lab/blob/main/Screenshot%20from%202025-12-05%2017-34-19.png)


## step 9. — Create EKS Cluster

## If eksctl is not installed in your local machine.Please install first eksctl in your local machine.
4.1 Create EKS cluster (simple command)
```bash
eksctl create cluster --name node-eks --region ap-south-1 --nodes 2
```
* This will take 10–15 minutes.
-------------------------
![app screenshot](https://github.com/jasimn/eks-lab/blob/main/Screenshot%20from%202025-12-05%2017-36-18.png)

## STEP 10 — Configure kubectl
-------------------------
```bash
aws eks update-kubeconfig --name node-eks --region ap-south-1
```

Verify access:
```bash
kubectl get nodes
```

-------------------------
## STEP 11 — Create Kubernetes Deployment + Service
-------------------------
11.1 Create deployment YAML
```bash
vim deployment.yaml
```

Paste:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeapp-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp
    spec:
      containers:
      - name: nodeapp
        image: <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mynodeapp:1.0
        ports:
        - containerPort: 3000
```
Apply:
```bash
kubectl apply -f deployment.yaml
```

11.2 Create Service (LoadBalancer)
```bash
vim service.yaml
```
Paste:
```bash
apiVersion: v1
kind: Service
metadata:
  name: nodeapp-service
spec:
  type: LoadBalancer
  selector:
    app: nodeapp
  ports:
  - port: 80
    targetPort: 3000
```

Apply:
```bash
kubectl apply -f service.yaml
```
-------------------------
## STEP 12 — Access the Application
-------------------------

Check external load balancer URL:
```bash
kubectl get svc nodeapp-service
```
Output:

* EXTERNAL-IP = a984b1569d88d45dc9de2ed2e1e8327e-884567909.us-east-1.elb.amazonaws.com

Now we can open our app on this url:
```bash
http://a984b1569d88d45dc9de2ed2e1e8327e-884567909.us-east-1.elb.amazonaws.com
```
🎉 Your Node.js app is now running on EKS!
  

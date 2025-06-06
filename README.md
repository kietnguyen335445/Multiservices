# Deploying Microservices on Kubernetes via Jenkins CI/CD Pipeline

## 📌 Project Overview

This project demonstrates the end-to-end automation of deploying microservices into a Kubernetes cluster (Amazon EKS) using Jenkins CI/CD pipelines. The entire setup, from infrastructure provisioning to automated builds and deployment, is hosted on AWS using a base EC2 instance.

## Services:
- Email service
- Frontend
- Cart service
- Product Catalog
- Adservice
- Payment service
- Currency service
- Shipping service
- Recommendation service
- Checkout service
---

## 🚀 Tech Stack

- **Cloud**: AWS (EC2, EKS, IAM, VPC)
- **CI/CD**: Jenkins
- **Containerization**: Docker
- **Orchestration**: Kubernetes (EKS)
- **Source Control**: GitHub

---

## 🏗️ Infrastructure Setup

### 1. EC2 Base Server
- **AMI**: Ubuntu 24
- **Instance Type**: t2.large
- **Storage**: 30 GB
- **Open Ports**: 22, 80, 443, 8080
  

### 2. IAM User
- Go to aws console and search for IAM and then click on user>>create user
- Created IAM user with necessary policies for EKS, EC2, IAM, CloudFormation:
  - AmazonEC2FullAccess
  - AmazonEKSClusterPolicy
  - AmazonEKSWorkerNodePolicy
  - AWSCloudFormationFullAccess
  - IAMFullAccess
  ![aws1](https://github.com/user-attachments/assets/d06e6662-43bd-4668-b699-cdb9e31c728a)
- Create an Inline policy for eks
    ```bash
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
    }

- Create a access keys and secret keys to work with awscli in base ec2 by go to User>>Create acess key>>Command Line Interface. Then download secret and acess keys file
- Generated and configured access/secret keys on EC2 using `aws configure`. Use information in the earlier file downloaded
 
---

## ⚙️ Kubernetes Cluster (EKS)

### Commands Used:
```bash
eksctl create cluster --name=EKS-1 --region=ap-southeast-1 --zones=ap-southeast-1a,ap-southeast-1b --without-nodegroup
```
![aaws2](https://github.com/user-attachments/assets/663eaec7-22cc-479f-8ced-64fe96917f27)
```
eksctl utils associate-iam-oidc-provider --region ap-southeast-1 --cluster EKS-1 --approve
```
```
eksctl create nodegroup --cluster=EKS-1 --region=ap-southeast-1 --name=node2 --node-type=t3.medium --nodes=3 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=my-ec2-keypair --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```
![aws3](https://github.com/user-attachments/assets/960f1132-da3b-461d-b17f-ccd45d9cdeb3)
```
aws eks update-kubeconfig --region ap-southeast-1 --name EKS-1
```
![EKS](https://github.com/user-attachments/assets/062c355a-0fbf-4654-980d-2ab3f1f050f2)


![node](https://github.com/user-attachments/assets/ad42db17-1b3d-4bfe-b5f6-26172e96ede7)


### Kubernetes Resources
- Namespace: webapps
  - Create a namesapce:
    ```bash
    kubectl create namespace webapps
    kubectl get namespaces
    ```
- ServiceAccount: jenkins
  - Creating Service Account:
    ```bash
    nano svc-acc.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: jenkins
      namespace: webapps
    ```
    ```bash
    kubectl apply -f svc-acc.yaml
    ```
- Role & RoleBinding for RBAC
  - Create role :
    ```bash
    nano app-role.yaml
    ```
    ```bash
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: app-role
      namespace: webapps
    rules:
    - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    ```
    ```bash
    kubectl -f apply app-role.yaml
    ```
  - Bind the role to service account:
    ```bash
    nano role-bind.yaml
    ```
    ```bash
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: app-rolebinding
      namespace: webapps 
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: app-role 
    subjects:
    - namespace: webapps 
      kind: ServiceAccount
      name: jenkins 
    ```
    ```bash
    kubectl -f apply role-bind.yaml
    ```
- Secret for service account token
  ```bash
  nano secret.yaml
  ```
  ```bash
  apiVersion: v1
  kind: Secret
  type: kubernetes.io/service-account-token
  metadata:
    name: mysecretname
    annotations:
    kubernetes.io/service-account.name: jenkins
  ```
  ```bash
  kubectl apply -f secret.yaml -n webapps
  ```
- Run the following command to get a secret token
  ```bash
  kubectl describe secret mysecretname -n webapps
  ```
## 🔧 Essential Tools Installed

- Installed via shell script:
  ```bash
  nano install.sh
  ```
  ```bash
  #!/bin/bash
  sudo apt update -y
  wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
  echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
  sudo apt update -y
  sudo apt install temurin-17-jdk -y
  /usr/bin/java --version

  #install jenkins
  curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt-get update -y
  sudo apt-get install jenkins -y
  sudo systemctl start jenkins
  sudo systemctl status jenkins

  #install docker
  sudo apt-get update
  sudo apt-get install docker.io -y
  sudo usermod -aG docker ubuntu
  sudo usermod -aG docker jenkins
  newgrp docker
  sudo chmod 777 /var/run/docker.sock
  sudo systemctl restart jenkins


  # Install AWS CLI 
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  sudo apt-get install unzip -y
  unzip awscliv2.zip
  sudo ./aws/install


  # Install kubectl
  sudo apt update
  sudo apt install curl -y
  curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
  kubectl version --client

  # Install eksctl
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  eksctl version
  ```
- Run the command:
  ```bash
  bash install.sh
  ```
# 🧪 Jenkins Setup
- Go to browser and type public_ip:8080
![jenkin](https://github.com/user-attachments/assets/12410ce8-44b2-4efd-a526-6ec16b6f9ed5)
- Run the command to get a password:
  ```bash
  cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Then choose the "Install the suggested plugins" option
Installed plugins:

- Docker

- Kubernetes

- Multibranch Scan Webhook Trigger

  ![plugin](https://github.com/user-attachments/assets/74ba0f94-92b1-4d03-8026-38899d22dedc)
## Configured credentials:
- Setup docker credentials
  - Go to jenkins →manage jenkins →credentials →global →add credentials
  ![docker-cre](https://github.com/user-attachments/assets/933a08a4-e5a9-443f-ae2a-469fed26c788)

- Setup github credentials use token
  - Go to github -> setting -> develop setting -> personal access token -> token(classic) -> generate new
  ![github](https://github.com/user-attachments/assets/58fd0efa-60a3-4bae-b0f4-1d00c88567d2)

- Kubernetes Token (id: k8-token)
  - Run the following command to get kubernetes token:
    ```bash
       kubectl describe secret mysecretname -n webapps
    ```
    ![k8-token](https://github.com/user-attachments/assets/cd6f5150-95ed-4ab5-9ba4-331a244d3fe5)

## Create jenkins pipeline
- Click on New item and chose multibranch pipeline
- Now choose git from add source option and paste the project repo url there
 ![jenkin2](https://github.com/user-attachments/assets/fdbe22f7-fd1f-4a95-8089-9012c78acd68)
![jenkin3](https://github.com/user-attachments/assets/c32923f0-ed0f-4ff3-8a37-b1100589b099)

- Configured GitHub webhook for CI/CD triggering:
  - Go to project repo and click on setting>>webhooks>>Add a webhook
   ![webhook](https://github.com/user-attachments/assets/6915ed4e-d0f1-4ff2-9748-f17f5b4a893a)
  - In Url section type
```bash
http://<public_ip>:8080/multibranch-webhook-trigger/invoke?token=microservice
```
![build](https://github.com/user-attachments/assets/d9960185-6ed7-44d7-b808-e9f8486c98b1)
  - Copy the Load-balancer url and paste it on browser, using http


![online-boutique-frontend-1](https://github.com/user-attachments/assets/8991ecd7-04b6-4b76-9a89-f37649a906f9)

![online-boutique-frontend-2](https://github.com/user-attachments/assets/0b918904-56cb-4d65-9ae7-d9aa830c6227)



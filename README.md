# Deploying 10 Microservices on Kubernetes via Jenkins CI/CD Pipeline

## 📌 Project Overview

This project demonstrates the end-to-end automation of deploying 10 microservices into a Kubernetes cluster (Amazon EKS) using Jenkins CI/CD pipelines. The entire setup, from infrastructure provisioning to automated builds and deployment, is hosted on AWS using a base EC2 instance.

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
- Created IAM user with necessary policies for EKS, EC2, IAM, CloudFormation.
- Generated and configured access/secret keys on EC2 using `aws configure`.

---

## ⚙️ Kubernetes Cluster (EKS)

### Commands Used:
```bash
eksctl create cluster --name=EKS-1 --region=ap-southeast-1 --zones=ap-southeast-1a,ap-southeast-1b --without-nodegroup

eksctl utils associate-iam-oidc-provider --region ap-southeast-1 --cluster EKS-1 --approve

eksctl create nodegroup --cluster=EKS-1 --region=ap-southeast-1 --name=node2 --node-type=t3.medium --nodes=3 ...

aws eks update-kubeconfig --region ap-southeast-1 --name EKS-1
```
### Kubernetes Resources
- Namespace: webapps

- ServiceAccount: jenkins

- Role & RoleBinding for RBAC

- Secret for service account token

## 🔧 Essential Tools Installed
Installed via shell script:

- Docker

- Jenkins

- AWS CLI

- Java 17 (Temurin)

- kubectl

- eksctl

# 🧪 Jenkins Setup
Installed plugins:

- Docker

- Kubernetes

- Multibranch Scan Webhook Trigger

## Configured credentials:

- DockerHub (id: docker)

- GitHub (id: github)

- Kubernetes Token (id: k8-token)

## Created multibranch pipeline for automatic builds and deployments.

- Configured GitHub webhook for CI/CD triggering:
```bash
http://<public_ip>:8080/multibranch-webhook-trigger/invoke?token=microservice
```
# 📦 Microservices Deployment
- 10 microservices pulled from GitHub repository.

- Each deployed into the webapps namespace on EKS.

- Services exposed via LoadBalancer for public access.



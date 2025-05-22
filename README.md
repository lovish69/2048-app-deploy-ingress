# 2048 Game Deployment on AWS EKS with Fargate + Ingress

This project demonstrates how to deploy a containerized 2048 game on a fully managed, serverless Kubernetes environment using **Amazon EKS with Fargate**, along with **Ingress Controller** to expose the application via a public URL.

---

## 🚀 Project Architecture

- **AWS EKS Cluster (Fargate)** – Fully managed, serverless compute for Kubernetes
- **Fargate Profiles** – Automatically provisions compute without managing nodes
- **Kubernetes Deployment** – Containerized 2048 game app deployed using `kubectl`
- **Ingress Controller** – NGINX Ingress used to route external traffic to the service
- **Load Balancer** – Automatically created via Ingress for external access

---

## 🧰 Tools & Services Used

| Tool            | Purpose                                       |
|-----------------|-----------------------------------------------|
| AWS EKS         | Managed Kubernetes cluster                    |
| Fargate         | Serverless pods (no EC2 management)           |
| Docker          | Containerized the 2048 game                   |
| kubectl         | Deployed app and services to EKS              |
| NGINX Ingress   | Routes external traffic to the service        |
| AWS IAM + OIDC  | Secure access via IAM Roles for Service Account |

---

## ✅ Features

- Fully serverless Kubernetes deployment (via AWS Fargate)
- Scalable and cost-efficient setup
- External access using Ingress and Load Balancer
- OIDC enabled for secure IAM role assumption by K8s service accounts

---

# Install EKS

Please follow the prerequisites doc before this.

## Install using Fargate

```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

## Delete the cluster

```
eksctl delete cluster --name demo-cluster --region us-east-1
```

![Screenshot (73)](https://github.com/user-attachments/assets/b0838aa0-280e-4b9c-89db-ef943cd92dee)

## Check if there is an IAM OIDC provider configured already

- aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4\n 

If not, run the below command

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

![Screenshot (75)](https://github.com/user-attachments/assets/055882cf-9fc4-46b0-af01-2e9020628e55)

# How to setup alb add on

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Deploy ALB controller

Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![Screenshot (77)](https://github.com/user-attachments/assets/0ef27f8c-9f47-445c-9e9a-c594839443f0)

![Screenshot (78)](https://github.com/user-attachments/assets/36731584-c417-4381-abf2-50b135126ab5)


## Deployment
![Screenshot (79)](https://github.com/user-attachments/assets/24415004-0f70-4116-b4c0-df8a3d3c72bf)




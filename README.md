# Deploying 2048 Game on Amazon EKS

This guide outlines the steps to deploy the 2048 game on Amazon EKS (Elastic Kubernetes Service). The deployment includes creating an EKS cluster, configuring Fargate profiles, deploying the game using Kubernetes manifests, and setting up an Application Load Balancer (ALB) for external access.

## Prerequisites

- AWS CLI installed and configured
- kubectl installed
- eksctl installed
- Helm installed

## 1. Create EKS Cluster

```bash
eksctl create cluster --name <cluster-name> --region <region>
```

# 2. Create kubeconfig File

```bash
aws eks --region <region> update-kubeconfig --name <cluster-name>
```

# 3. Create Fargate Profile

```bash
eksctl create fargateprofile --cluster <cluster-name> --name <fargate-profile-name> --namespace <namespace>
```

# 4. Deploy 2048 Game

```bash
kubectl apply -f 2048-game.yaml
```

# 5. Configure IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider --region <region> --cluster <cluster-name> --approve
```
# 6. Create IAM Role, Policy, and Service Account

```bash
# Create IAM Role and Policy
aws iam create-role --role-name <role-name> --assume-role-policy-document file://eks-role-trust-policy.json
aws iam put-role-policy --role-name <role-name> --policy-name <policy-name> --policy-document file://eks-role-policy.json

# Create Service Account and Associate IAM Role
eksctl create iamserviceaccount --region <region> --name <service-account-name> --namespace <namespace> --cluster <cluster-name> --attach-policy-arn arn:aws:iam::<account-id>:policy/<policy-name> --approve
```

# 7. Deploy ALB Controller

```bash
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=release-1.2"
helm repo add eks https://aws.github.io/eks-charts
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=<service-account-name>
```

# 8. Install Helm Chart for 2048 Game

```bash
helm install 2048-game ./2048-helm-chart
```
# 9. Verify Load Balancer

```bash
kubectl get ingress
```









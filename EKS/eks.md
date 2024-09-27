---
title: EKS
---
## EKS Documentation

### Deploying AWS EKS using terraform

## AWS EKS

AWS EKS is a managed Kubernetes service that simplifies the deployment, management, and scaling of containerized applications using Kubernetes.

## Terraform

Terraform is an infrastructure as code tool that allows you to define and provision infrastructure using a declarative configuration language.

## GitLab/GitHub Actions

GitLab/GitHub Actions can automate the deployment process through continuous integration and continuous deployment (CI/CD) pipelines.


## Prerequisites

1. **AWS Account**:Make sure you have an AWS account set up.
2. **Terraform Installed**:Download and install Terraform on your local machine.
3. **AWS CLI**:Install the AWS Command Line Interface (CLI) for managing AWS resources.
4. **kubectl**:Install kubectl, the command-line tool for Kubernetes.
5. **GitLab/GitHub Account**:You’ll need a repository to store your Terraform code.


## Step 1 : Set Up Your AWS Credentials

1. **Configure AWS CLI** :
```bash
aws configure
```
Enter your AWS Access Key, Secret Access Key, region, and output format.

## Step 2 : Create a Terraform Configuration File

2. **Directory Structure**:Create a new directory for your Terraform files:
```bash
mkdir eks-terraform
cd eks-terraform
```

1. **Main Configuration File (main.tf)**:Create a main.tf file and define your provider and resources.

### main.tf
```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
  required_version = ">= 0.12"
}

provider "aws" {
  region = "us-west-2" # Change to your desired region
}

resource "aws_eks_cluster" "my_cluster" {
  name     = "my-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn

  vpc_config {
    subnet_ids = aws_subnet.my_subnet.*.id
  }
}

resource "aws_iam_role" "eks_cluster_role" {
  name = "eks_cluster_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Principal = {
        Service = "eks.amazonaws.com"
      }
      Effect = "Allow"
      Sid    = ""
    }]
  })
}

resource "aws_subnet" "my_subnet" {
  vpc_id     = aws_vpc.my_vpc.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

output "cluster_endpoint" {
  value = aws_eks_cluster.my_cluster.endpoint
}

output "cluster_name" {
  value = aws_eks_cluster.my_cluster.name
}

```

3. **Define Additional Resources**: You may need to add VPC and subnet resources for your cluster, along with EKS node groups for worker nodes. For example:

```
resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "eks_subnet" {
  count             = 2
  vpc_id            = aws_vpc.eks_vpc.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = element(data.aws_availability_zones.available.names, count.index)
}

resource "aws_eks_node_group" "my_node_group" {
  cluster_name    = aws_eks_cluster.my_cluster.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.node_group_role.arn
  subnet_ids      = aws_subnet.eks_subnet.*.id

  scaling_config {
    desired_size = 2
    max_size     = 3
    min_size     = 1
  }

  # Optional: Specify instance type
  instance_type = "t3.medium"
}
```

## Step 3 : Initialize Terraform
```
terraform init
```
## Step 4 : Plan your deployment
```
terraform plan
```
## Step 5 : Apply the configuration
```
terraform apply
```
## Step 6 : Configure kubectl
1. After the cluster is created, configure kubectl to use the EKS cluster:

```bash
aws eks update-kubeconfig --name my-cluster --region us-west-2
```
2. Verify that kubectl is configured correctly:
```bash
kubectl get svc
```
## Step 7 : Set Up CI/CD Pipeline with GitLab/GitHub

For GitHub Actions
1. Create a .github/workflows/ci.yml file in your repository:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.0.0

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        run: terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
2. Add AWS credentials to your GitHub repository secrets.

**For GitLab CI/CD**
1. Create a .gitlab-ci.yml file in your repository:
```yaml
stages:
  - deploy

deploy:
  stage: deploy
  image: hashicorp/terraform:latest
  script:
    - terraform init
    - terraform plan
    - terraform apply -auto-approve
  only:
    - main
  variables:
    AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
```
2. Add AWS credentials to your GitLab CI/CD variables.
## Step 7 : Testing the Deployment
1. Push your changes to the repository. This should trigger the CI/CD pipeline.
2. Monitor the pipeline execution in GitHub Actions or GitLab CI/CD interface.
3. After a successful run, you should have an EKS cluster deployed.

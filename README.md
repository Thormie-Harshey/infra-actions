
# GitOps with GitHub Actions: EKS Infrastructure

This repository contains the Infrastructure as Code (IaC) for a project that automates the provisioning of an AWS EKS (Elastic Kubernetes Service) cluster and its supporting resources using Terraform. The entire process is orchestrated by a CI/CD pipeline built with GitHub Actions.

This infrastructure is designed to host a contact form application, which is managed in a separate repository.

## **Project Overview**

The goal of this project is to implement a GitOps workflow where all infrastructure configurations are version-controlled in this repository. A GitHub Actions pipeline automatically provisions and tears down the AWS environment based on changes to the Terraform code.


## **Related Repository**

**Application Code:** The application that will be deployed to this EKS cluster is located in the   **App Code Repo:** [appcode-contactform](https://github.com/Thormie-Harshey/appcode-contactform)  
The app repo builds and deploys a PHP application to the EKS cluster created by this repository.

## **Architecture and Technologies**

The infrastructure architecture is defined as follows:
| Category        | Tools / Services |
|----------------|------------------|
| **Cloud** | AWS |
| **Networking** | VPC, Subnets (Public & Private), Route Tables, NAT Gateway |
| **Load Balancing** | NGINX Ingress Controller (deployed as part of the EKS setup) |
| **Compute** | EC2 (Auto Scaling Group, Launch Templates) |
| **Container Orchestration** |  EKS Cluster |
| **Container Registry** |  ECR Repository |
| **Container Orchestration** |  EKS Cluster |
| **CI/CD** | GitHub Actions |
| **Infrastructure as Code (IaC)** | Terraform |


## **Getting Started**

To successfully use this repository, you must have the following configured in your GitHub repository secrets and variables:

#### **GitHub Repository Variables**

1.  `AWS_REGION`: The AWS region where resources will be deployed (e.g., `us-east-1`).
2.  `S3_BACKEND_BUCKET`: The name of the S3 bucket to store the Terraform state file.
3.  `ECR_REPOSITORY`: The name of the ECR repository to be created.

#### **GitHub Repository Secrets**

1.  `AWS_ACCESS_KEY_ID`: Your AWS access key.
2.  `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.

> **Known limitation:** this pipeline authenticates with long-lived AWS access keys stored as GitHub secrets, rather than GitHub OIDC. Other repos in this profile (`petclinic-platform`, `php-web-app-iac`) use a GitHub-OIDC-trusted IAM role instead, which avoids storing AWS credentials in GitHub entirely. This repo predates that pattern and hasn't been migrated yet — doing so would mean adding an OIDC-trusted IAM role to the Terraform here and switching `iac.yml` to request a short-lived token via `permissions: id-token: write` instead of reading these secrets.

## **Usage**
#### **1. Deploying the Infrastructure**

The deployment workflow is defined in `.github/workflows/iac.yml`. It is triggered on a `push` to the `infra-actions` branch.

* Simply push your Terraform changes to the `infra-actions` branch.
* The GitHub Actions workflow will automatically run `terraform init`, `terraform plan`, and `terraform apply`.
* You can monitor the progress of the deployment in the "Actions" tab of your repository.

#### **2. Destroying the Infrastructure**

When it comes to tearing down the entire environment, a manual trigger is required. This manual trigger is known as (`workflow_dispatch`).

* Navigate to the "Actions" tab.
* Select the `Deploy/Destroy Infrastructure` workflow.
* Click the "Run workflow" button.
* In the dropdown menu, select the `destroy` option to trigger the destruction process.

This process will first run a cleanup script to delete the NGINX Ingress controller and empty the ECR, preventing pipeline failures before Terraform tears down the rest of the resources.

---
### **Outputs**

The Terraform deployment provides crucial outputs that are required for the application deployment pipeline. You will need to copy these values from the workflow logs and configure them as secrets/variables in the **application repository**. Outputs such as:

* `EKS_CLUSTER_NAME`: The name of the EKS cluster.
* `REGISTRY`: The URL of the ECR registry.
* `KUBECONFIG`: The kubeconfig file content required to connect to the EKS cluster.


## The Key Concepts

The key concepts highlighted when taking on this particular project include:

- **1. Terraform plan & apply automation** with `input=false` and `parallelism=1`:
The`input=false` flag ensures non-interactive execution between you and the Terraform pipeline.
-   A subsequent `terraform apply -auto-approve` step uses the saved `planfile` to apply the changes, ensuring that the deployed infrastructure matches the plan exactly.
-   `parallelism=1` controls how many resources Terraform will create/update at the same time. The default is 10, but here in this project, it’s set to 1, meaning Terraform will work on one resource at a time, and the reason for this is to reduce the risk of dependency issues and avoid race conditions.
- **Planfile reuse** for consistent deployments: `terraform plan` creates an execution plan and saves it to a planfile, which can be reused in subsequent jobs within this terraform workflow
- **Secrets Management** via GitHub Secrets


## **Outcomes and Results**

-   **Fully Automated Pipeline:** A CI/CD pipeline was successfully implemented, demonstrating the power of GitHub Actions for a complete GitOps workflow.
-   **Reproducible Infrastructure:** The use of Terraform and IaC ensures that the entire AWS environment is reproducible and can be spun up or down with a single command.

# DevSecOps Deployment: OpenAI Chatbot UI on EKS with Jenkins & Terraform

![Chatbot UI Architecture](https://imgur.com/MdxoqmL.png)

## Overview

This project walks through deploying a secure, production-grade OpenAI-powered chatbot UI using a modern DevSecOps toolchain. We used:

* **Docker** to containerize the app
* **Jenkins** to drive our CI/CD pipelines
* **EKS** (Amazon's Kubernetes service) to manage and scale deployments
* **Terraform** to provision infrastructure-as-code

Security is woven into every stage, embracing the mythical security-by-design - shift-left principles, automated scans and hardened delivery pipelines are all included.

### What’s the Chatbot?

An OpenAI-powered conversational interface designed to simulate natural language dialogue. It's always on, scales effortlessly and provides contextual, human-like responses using OpenAI’s APIs.

### Why Build It This Way?

* **24/7 Responsiveness:** Fully automated support layer that never sleeps.
* **Scalability:** Handle thousands of user requests with Kubernetes auto-scaling.
* **Security-First:** DevSecOps practices like OWASP scans, SonarQube quality gates, and Trivy image scanning aim to catch issues early.

---

## Quick Summary of Stack

* **Containerization:** Docker ensures consistent runtime environments
* **Orchestration:** Kubernetes (EKS) automates deployments, scaling, and recovery
* **CI/CD:** Jenkins automates testing, building, scanning, and releasing
* **Security:** SonarQube, OWASP Dependency-Check, Trivy, and GitHub Secrets
* **Infra Provisioning:** Terraform handles backend infra with locking via DynamoDB

---

## Jenkins Setup & Chatbot UI Deployment

**GitHub Repo**: [Chatbot UI](https://github.com/KenB773/ChatbotUIJenkins/)

### Step 1: Provision Jenkins with Terraform

Clone the repo and prepare the Terraform backend:

```bash
git clone https://github.com/KenB773/ChatbotUIJenkins/
cd Jenkins-Server-TF
```

Make sure to:

* Set up S3 and DynamoDB for remote state
* Create AWS access keys
* Install Terraform and AWS CLI

Initialize with:

```bash
terraform init
terraform apply -var-file=variables.tfvars --auto-approve
```

Result: An EC2 instance labeled `Jenkins-server` is up and ready.

### 🔧 Step 2: Jenkins Web UI Setup

1. Visit EC2\_IP:8080 and unlock Jenkins with the admin password.
2. Install default plugins.
3. Set up a Jenkins user.
4. Install additional tools: JDK, NodeJS, Docker, SonarQube Scanner, AWS CLI, Kubernetes CLI, etc.
5. Configure tool installs under **Manage Jenkins → Global Tool Configuration**.
6. Add credentials for DockerHub, GitHub, AWS, and SonarQube under **Manage Jenkins → Credentials**.

---

## Step 3: Pipeline for Secure Build & Docker Deployment

Set up your first Jenkins pipeline:

* Use "Pipeline from SCM"
* Pull `Jenkinsfile` from GitHub repo

Stages include:

* SonarQube scanning
* OWASP dependency check
* Trivy filesystem and image scans
* Docker build & push
* OpenAI Chatbot container deployment on port 3000

Once deployed, access the UI and provide your OpenAI API key via the interface.

---

## Step 4: Create EKS Cluster

Use another Jenkins pipeline to deploy EKS:

* Pull Jenkinsfile for EKS cluster creation
* On success, run:

```bash
aws eks update-kubeconfig --name <cluster_name> --region <region>
```

* Save `.kube/config` file and upload to Jenkins Credentials

---

## Step 5: Deploy Chatbot UI on EKS

Append the following to your Jenkinsfile:

```groovy
stage('Deploy to kubernetes') {
  steps {
    withAWS(credentials: 'aws-key', region: 'us-east-1') {
      script {
        withKubeConfig(credentialsId: 'k8s') {
          sh 'kubectl apply -f k8s/chatbot-ui.yaml'
        }
      }
    }
  }
}
```

Trigger a new build and verify deployment:

```bash
kubectl get all
```

Get the external load balancer URL and test the chatbot.

---

## Step 6: Clean Up

* Destroy EKS via Jenkins pipeline
* Takedown Jenkins resources:

```bash
terraform destroy -var-file=variables.tfvars --auto-approve
```

---

This project showcases how to secure, automate, and deploy a cloud-native AI service using best practices across DevOps, MLOps, and infrastructure-as-code.

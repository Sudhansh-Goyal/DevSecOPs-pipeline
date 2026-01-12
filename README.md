# DevSecOps CI/CD Pipeline

A complete **DevSecOps CI/CD pipeline** with integrated security scanning, automated testing, and multi-environment deployment using **Jenkins**, **ArgoCD**, and **AWS services**.

---

##  Pipeline Features

* Security scanning at every stage:

  * SAST (Static Application Security Testing)
  * DAST (Dynamic Application Security Testing)
  * Dependency vulnerability checks
  * Container image scanning
* Multi-environment deployments:

  * **Development:** AWS EC2
  * **Staging:** Kubernetes (MicroK8s)
  * **Production:** AWS Lambda
* GitOps workflow using **ArgoCD** and **Bitnami Sealed Secrets**
* Automated quality gates and policy enforcement
* Comprehensive reporting and notifications

---

##  Infrastructure Overview

### Core Components

| Component            | Description                                                               |
| -------------------- | ------------------------------------------------------------------------- |
| **Jenkins Server**   | CI/CD orchestration running on AWS EC2 (t2.large)                        |
| **Gitea Server**     | Self-hosted Git repository with webhook integration (same EC2 as Jenkins) |
| **MicroK8s Cluster** | Kubernetes-based staging environment on AWS EC2 (t2.medium)               |
| **SonarQube**        | Static code analysis and quality gate enforcement                         |
| **ArgoCD**           | GitOps-based continuous deployment controller                             |
| **AWS Lambda**       | Serverless production deployment target                                   |
| **Amazon S3**        | Artifact and build output storage                                         |

---

##  Deployment Environments

### 1 Development Environment

* **Branch Pattern:** `feature/*`
* **Deployment Target:** AWS EC2 (direct deployment)
* **Validations:**

  * Build and unit tests
  * Integration testing
  * Dependency vulnerability scanning

---

### 2️ Staging Environment

* **Branch Pattern:** `PR*`
* **Deployment Target:** Kubernetes (MicroK8s) via GitOps
* **Deployment Tool:** ArgoCD
* **Validations:**

  * Container image scanning (Trivy)
  * OWASP ZAP DAST scanning
  * Kubernetes manifest validation

---

### 3 Production Environment

* **Branch Pattern:** `main`
* **Deployment Target:** AWS Lambda
* **Approval:** Manual approval required
* **Validations:**

  * Final security checks
  * AWS Lambda function health checks
  

---

##  Security Integration

Security is embedded at every stage of the pipeline:

* **SAST:** SonarQube with enforced quality gates
* **Dependency Scanning:** OWASP Dependency Check
* **Container Scanning:** Trivy
* **DAST:** OWASP ZAP against staging endpoints
* **Secrets Management:** Bitnami Sealed Secrets (Git-safe encrypted secrets)

---


---

##  Prerequisites

### Infrastructure Requirements

* AWS EC2 (t2.medium) for Jenkins & Gitea
* MicroK8s cluster with:

  * ArgoCD installed
  * Bitnami Sealed Secrets controller
* AWS account with required IAM roles
* DockerHub registry access

---

### Jenkins Plugins

* NodeJS Plugin (v24.6.0)
* SonarQube Scanner (v7.2.0)
* OWASP Dependency Check (v12.1.2)
* AWS Pipeline Steps
* Docker Pipeline
* Slack Notification Plugin

---

##  Security Tools Stack

| Tool                       | Purpose                                       |
| -------------------------- | --------------------------------------------- |
| **SonarQube**              | SAST and code quality analysis                |
| **Trivy**                  | Container vulnerability scanning              |
| **OWASP ZAP**              | Dynamic application security testing          |
| **OWASP Dependency Check** | Third-party dependency vulnerability analysis |

---

##  Reporting & Notifications

* Jenkins build reports
* SonarQube quality gate dashboards
* Security scan summaries
* SNS Service configured for notifications for pipeline status and approvals

---

##  Summary

This DevSecOps CI/CD pipeline ensures:

* Continuous integration with security-first mindset
* Automated and controlled deployments across environments
* Strong governance using GitOps and quality gates
* Scalable, production-ready AWS infrastructure

---

*Designed for real-world DevSecOps practices and cloud-native deployments.*

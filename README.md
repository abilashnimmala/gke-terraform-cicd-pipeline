# Production-Ready GitOps Evolution: GKE, ArgoCD & Terraform

[![Terraform](https://img.shields.io/badge/IaC-Terraform-623CE4?logo=terraform)](https://www.terraform.io/)
[![GKE](https://img.shields.io/badge/K8s-GKE-4285F4?logo=google-cloud)](https://cloud.google.com/kubernetes-engine)
[![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-EF7B4D?logo=argo)](https://argoproj.github.io/cd/)
[![GitHub Actions](https://img.shields.io/badge/CI-GitHub_Actions-2088FF?logo=github-actions)](https://github.com/features/actions)

An end-to-end DevOps ecosystem demonstrating industrial-standard deployment patterns. This project automates the provisioning of a Google Kubernetes Engine (GKE) cluster, configures secure CI/CD pipelines using Workload Identity Federation, and manages application lifecycles via ArgoCD and Helm.

## 🏗️ Architecture

```text
                                     +--------------------------+
                                     |      Custom Domain       |
                                     |  dev.abilashnimmala.in   |
                                     +------------+-------------+
                                                  |
                                          [ HTTPS / Port 443 ]
                                                  |
                                     +------------v-------------+
                                     |   NGINX Ingress Controller|
                                     |  (Managed by cert-manager)|
                                     +------------+-------------+
                                                  |
       +------------------------------------------+------------------------------------------+
       |                                   GKE Cluster                                       |
       |  +-----------------------+      +------------------------+      +----------------+  |
       |  |    ArgoCD Manager     | <----+   Artifact Registry    | <----+ GitHub Actions |  |
       |  | (GitOps Controller)   |      |   (Docker Images)      |      |  (CI Pipeline) |  |
       |  +-----------+-----------+      +------------------------+      +--------^-------+  |
       |              |                                                           |          |
       |  +-----------v-----------+                                      +--------+-------+  |
       |  |  Helm-based App Pods  |                                      | Workload Ident.|  |
       |  |  (Scaling & Self-Heal)|                                      | Federation OIDC|  |
       |  +-----------------------+                                      +----------------+  |
       +-------------------------------------------------------------------------------------+
```

## 🛠️ Tech Stack

*   **Infrastructure:** Terraform (HCL), Google Cloud Platform (GCP)
*   **Orchestration:** Google Kubernetes Engine (GKE), Helm v3
*   **CI/CD:** GitHub Actions, ArgoCD (GitOps)
*   **Security:** GCP Workload Identity Federation (Keyless Auth), cert-manager (Let's Encrypt)
*   **Networking:** NGINX Ingress Controller, Cloud DNS
*   **Containerization:** Docker, Google Artifact Registry

## 🔄 Deployment Workflow

1.  **Infrastructure Phase:** Terraform provisions VPC, GKE (Standard/Autopilot), and Artifact Registry.
2.  **Security Handshake:** GitHub Actions authenticates to GCP via **Workload Identity Federation (OIDC)**—eliminating the need for long-lived Service Account JSON keys.
3.  **CI Pipeline:** On every push to `main`, GHA builds the Docker image, tags it with the short SHA, and pushes it to the Google Artifact Registry.
4.  **CD (GitOps) Phase:** ArgoCD monitors the `simple-app-chart` directory. Upon detecting a change (or manual sync), it pulls the latest Helm chart and reconciles the state of the GKE cluster.
5.  **Traffic Flow:** NGINX Ingress receives requests for `dev.abilashnimmala.in`. Cert-manager validates the ACME challenge with Let's Encrypt to provide an automated SSL certificate.

## 🚀 Key Features

*   **Zero-Trust Security:** Implementation of Workload Identity Federation for GHA-to-GCP communication.
*   **GitOps Maturity:** Automated drift detection and reconciliation using ArgoCD.
*   **SSL Automation:** Fully automated certificate issuance and renewal via cert-manager and Let's Encrypt.
*   **Dynamic Scaling:** GKE cluster configured with Horizontal Pod Autoscaler (HPA) and Cluster Autoscaler.
*   **Modular IaC:** Reusable Terraform modules for VPC and GKE.

## 📦 Setup Instructions

### Prerequisites
*   Google Cloud SDK installed and authenticated.
*   Terraform >= 1.5.0
*   A registered domain (e.g., on GoDaddy/Namecheap) pointed to Cloud DNS.

### 1. Provision Infrastructure
```bash
cd terraform
terraform init
terraform apply -var-file="terraform.tfvars"
```

### 2. Configure Cluster Access
```bash
gcloud container clusters get-credentials $(terraform output -raw kubernetes_cluster_name) --region $(terraform output -raw region)
```

### 3. Bootstrap ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 💰 Cost Optimization Strategy

To keep the project budget-friendly for production testing:
*   **Preemptible/Spot VMs:** Used for the GKE node pool to reduce compute costs by up to 60-80%.
*   **Ephemeral Environments:** Integrated `destroy.sh` and `create_all.sh` scripts to tear down the entire stack during non-working hours, preserving only the Terraform state.
*   **Artifact Lifecycle:** Configured Artifact Registry to delete images older than 30 days.

## 🧠 Lessons Learned

*   **DNS Propagation:** Handling the propagation lag between Cloud DNS and the domain registrar during Ingress setup.
*   **ArgoCD & Helm Interop:** Fine-tuning the `values.yaml` injection within ArgoCD Application manifests.
*   **OIDC Complexity:** Understanding the security benefits of Workload Identity Pools over traditional Service Account Keys.

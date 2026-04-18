# 🚀 Cloud-Native Task Manager – Complete DevOps Platform

A production-grade **Task Manager** web application demonstrating end-to-end **DevOps** practices. Built with **Flask + PostgreSQL**, containerized with **Docker**, orchestrated with **Kubernetes (AWS EKS)**, automated via **Jenkins CI/CD**, and monitored with **Prometheus + Grafana**. Infrastructure provisioned with **Terraform** on AWS.

**Perfect for:** Interview preparation, portfolio demonstration, and learning modern DevOps workflows.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Project Workflow & Flow Diagram](#project-workflow--flow-diagram)
5. [Prerequisites](#prerequisites)
6. [Installation Guide for Amazon Linux 2023](#installation-guide-for-amazon-linux-2023)
7. [Local Deployment with Docker Compose](#local-deployment-with-docker-compose)
8. [AWS Infrastructure Setup with Terraform](#aws-infrastructure-setup-with-terraform)
9. [Jenkins CI/CD Pipeline Setup](#jenkins-cicd-pipeline-setup)
10. [Kubernetes Deployment](#kubernetes-deployment)
11. [Monitoring & Observability](#monitoring--observability)
12. [Troubleshooting Guide](#troubleshooting-guide)
13. [Security Best Practices](#security-best-practices)
14. [Contributing](#contributing)

---

## 📌 Project Overview

### What is this Project?

A **comprehensive end-to-end DevOps showcasing project** that demonstrates:

- **Full-stack Application Development:** Flask REST API backend + HTML/CSS/JavaScript frontend
- **Containerization:** Docker-based packaging for consistency across environments
- **Container Orchestration:** Kubernetes (EKS) for production-grade deployment
- **Infrastructure as Code:** Terraform for AWS resource provisioning
- **CI/CD Automation:** Jenkins pipeline for build, test, and deploy automation
- **Code Quality:** SonarQube integration for static code analysis
- **Observability:** Prometheus metrics collection and Grafana visualization
- **Security:** Industry-standard practices for Kubernetes, container security, and secrets management

### Key Features

✅ **Production-Ready Architecture** - Follows AWS and Kubernetes best practices
✅ **Fully Automated CI/CD** - From Git commit to EKS deployment
✅ **Infrastructure as Code** - Complete AWS infrastructure via Terraform
✅ **Comprehensive Monitoring** - Real-time metrics and dashboards
✅ **Security-First Design** - Secrets management, RBAC, and hardening
✅ **Scalable & Resilient** - Auto-scaling, health checks, and rolling updates
✅ **Complete Documentation** - Detailed setup and troubleshooting guides

---

## 🏗️ System Architecture

### 3-Tier Application Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     END USERS / CLIENTS                      │
└────────────────────────┬──────────────────────────────────────┘
                         │ HTTP/HTTPS
┌────────────────────────▼──────────────────────────────────────┐
│                    AWS ELB (Load Balancer)                     │
└────────────────────────┬──────────────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────────────┐
│                   AWS EKS (Kubernetes)                         │
│  ┌────────────────────────────────────────────────────────┐   │
│  │          Kubernetes Cluster (Multiple Nodes)          │   │
│  │  ┌──────────────────┐      ┌──────────────────────┐   │   │
│  │  │   Frontend Pod   │      │   Frontend Pod       │   │   │
│  │  │ (Nginx + Static) │      │ (Nginx + Static)     │   │   │
│  │  │   Port: 80/443   │      │   Port: 80/443       │   │   │
│  │  └──────────────────┘      └──────────────────────┘   │   │
│  │                                                        │   │
│  │  ┌──────────────────┐      ┌──────────────────────┐   │   │
│  │  │  Backend Pod #1  │      │  Backend Pod #2      │   │   │
│  │  │  (Flask API)     │      │  (Flask API)         │   │   │
│  │  │   Port: 8888     │      │   Port: 8888         │   │   │
│  │  └──────────────────┘      └──────────────────────┘   │   │
│  │           │                         │                 │   │
│  │           └─────────────┬───────────┘                 │   │
│  │                         │                             │   │
│  │              ┌──────────▼────────┐                    │   │
│  │              │  PostgreSQL Pod   │                    │   │
│  │              │  Port: 5432       │                    │   │
│  │              │  Persistent Volume│                    │   │
│  │              └───────────────────┘                    │   │
│  └────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│              Monitoring & Observability Stack                   │
│  ┌─────────────────┐   ┌──────────────┐   ┌──────────────┐    │
│  │  Prometheus     │──▶│   Grafana    │   │  AlertManager│    │
│  │  (Metrics)      │   │ (Dashboard)  │   │  (Alerts)    │    │
│  └─────────────────┘   └──────────────┘   └──────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | Purpose | Technology |
|-----------|---------|-----------|
| **Frontend** | Static web UI, user interface | Nginx, HTML, CSS, JavaScript |
| **Backend API** | REST API, business logic | Flask (Python), SQLAlchemy |
| **Database** | Persistent data storage | PostgreSQL 15 |
| **Orchestration** | Container orchestration, scaling | AWS EKS, Kubernetes |
| **Container Registry** | Store & distribute container images | Docker Hub |
| **CI/CD** | Automation of build, test, deploy | Jenkins |
| **Code Quality** | Static code analysis, security scanning | SonarQube |
| **Monitoring** | Metrics collection, visualization | Prometheus, Grafana |
| **Infrastructure** | Cloud resources provisioning | AWS (EKS, EC2, RDS, VPC), Terraform |

---

## 🛠️ Technology Stack

### Backend Stack
```
Language:       Python 3.11
Framework:      Flask 3.0.0
ORM:            SQLAlchemy 2.0.25
Database:       PostgreSQL 15
App Server:     Gunicorn 21.2.0
CORS:           Flask-CORS 4.0.0
Dependencies:   python-dotenv, psycopg2-binary
```

### DevOps & Infrastructure
```
Containerization:    Docker & Docker Compose
Orchestration:       Kubernetes (AWS EKS)
Cloud Provider:      Amazon Web Services (AWS)
IaC Tool:           Terraform 1.x
CI/CD Platform:     Jenkins 2.x
Registry:           Docker Hub
```

### Frontend Stack
```
Server:         Nginx (Reverse Proxy)
UI:             HTML5, CSS3, JavaScript (Vanilla)
Features:       Dashboard, Task Management, User Auth, Analytics
```

### Monitoring & Observability
```
Metrics:        Prometheus
Visualization:  Grafana
Export:         Prometheus node-exporter, kube-state-metrics
Alerts:         Prometheus AlertManager
```

### Code Quality & Security
```
Static Analysis:    SonarQube
Linting:           Built-in code quality checks
Security Scanning:  SonarQube SAST
```

---

## 🔄 Project Workflow & Flow Diagram

### End-to-End Deployment Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPER WORKFLOW                            │
└─────────────────────────────────────────────────────────────────┘

   Step 1: Code Commit
   ─────────────────
   Developer writes code → Commits → Pushes to GitHub
                                         │
                                         ▼
   ┌─────────────────────────────────────────────────────────────┐
   │         Jenkins CI/CD Pipeline Triggered                     │
   └─────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 1: Source Code Checkout              │
        │   └─ Git pull latest code                    │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 2: Code Quality Analysis            │
        │   └─ SonarQube static analysis              │
        │   └─ Quality gate check (Pass/Fail)         │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │ (if passes)
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 3: Docker Build                    │
        │   ├─ Build backend Docker image             │
        │   ├─ Build frontend Docker image            │
        │   └─ Tag with version + build number        │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 4: Container Testing                │
        │   └─ Spin up with Docker Compose            │
        │   └─ Run HTTP health checks                 │
        │   └─ Verify app connectivity                │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │ (if tests pass)
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 5: Push to Registry                 │
        │   └─ Login to Docker Hub                    │
        │   └─ Push backend image                     │
        │   └─ Push frontend image                    │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 6: Deploy to Kubernetes (EKS)      │
        │   ├─ Create namespace                       │
        │   ├─ Apply secrets                          │
        │   ├─ Deploy PostgreSQL with PVC             │
        │   ├─ Deploy backend deployment              │
        │   ├─ Deploy frontend deployment             │
        │   ├─ Setup ingress rules                    │
        │   └─ Wait for deployment rollout            │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   STAGE 7: Deploy Monitoring Stack          │
        │   ├─ Setup Prometheus                       │
        │   ├─ Setup Grafana                          │
        │   ├─ Configure dashboards                   │
        │   └─ Enable alerting                        │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              │
                              ▼
        ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
        │   ✅ DEPLOYMENT COMPLETE                   │
        │   Application accessible via Ingress URL    │
        │   Metrics visible in Grafana Dashboard      │
        └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

### Application Data Flow

```
┌──────────────┐
│   User       │
│  (Browser)   │
└──────┬───────┘
       │ HTTP Request (port 80)
       ▼
┌──────────────────────┐
│   Nginx (Frontend)   │
│  • Static assets     │
│  • Reverse proxy     │
│  • Load balancer     │
└──────┬───────────────┘
       │ HTTP Request (port 8888)
       ▼
┌──────────────────────┐
│  Flask Backend API   │
│  • Route handlers    │
│  • Business logic    │
│  • Authentication    │
└──────┬───────────────┘
       │ SQL Queries
       ▼
┌──────────────────────┐
│  PostgreSQL Database │
│  • Users table       │
│  • Tasks table       │
│  • Persistent data   │
└──────────────────────┘

Response Flow (Reverse):
Database → Backend API → Nginx → Browser JSON/HTML
```

---

## 📋 Prerequisites

### Local Development Machine

```bash
# Required tools
✓ Git 2.30+
✓ Docker 20.10+
✓ Docker Compose 2.0+
✓ Python 3.9+
✓ kubectl 1.24+
✓ Terraform 1.0+
✓ AWS CLI 2.0+
```

### AWS Account Requirements

- AWS Account with appropriate IAM permissions
- EC2 instance for Jenkins (t3.medium minimum)
- EKS cluster setup (or use Terraform to provision)
- ECR or Docker Hub for image registry
- S3 bucket for Terraform state storage

### Network Requirements

- Security group ingress for: SSH (22), Jenkins (8080), Kubernetes API (6443)
- NAT gateway or IGW for outbound traffic
- VPC with subnets for EKS cluster nodes

---

## 📦 Installation Guide for Amazon Linux 2023

This guide follows **Linux security and installation best practices** for Amazon Linux 2023.

### System Preparation

#### Step 1: Update System Packages (Required)

```bash
# Update package manager and install base utilities
sudo dnf update -y
sudo dnf install -y wget curl git htop tmux vim nano nc zip unzip

# Install development tools
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y gcc g++ make openssl-devel
```

**Why:** Ensures security patches and required utilities are installed.

---

#### Step 2: Install Docker with Proper Configuration (Best Practice)

```bash
# Install Docker from Amazon's ECR
sudo dnf install -y docker

# Add ec2-user to docker group (eliminates sudo requirement)
sudo usermod -aG docker ec2-user

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
docker run hello-world

# Configure Docker daemon (security hardening)
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "live-restore": true,
  "userland-proxy": false
}
EOF

# Reload Docker daemon
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**Best Practices Applied:**
- JSON logging for better log management
- overlay2 storage driver (modern & efficient)
- Live restore for container recovery
- Removed userland proxy (security)

---

#### Step 3: Install Docker Compose

```bash
# Download latest Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

**Why:** Allows simplified multi-container management for local testing.

---

#### Step 4: Install Python 3.11+ (Required for Flask)

```bash
# Install Python and essential packages
sudo dnf install -y python3.11 python3.11-devel python3.11-pip

# Set Python 3.11 as default
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1

# Upgrade pip and install virtualenv
python3 -m pip install --upgrade pip
python3 -m pip install virtualenv

# Verify
python3 --version
python3 -m pip --version
```

**Best Practices:**
- Using specific Python version (3.11)
- Virtual environment support for isolation
- Pip upgrade for security fixes

---

#### Step 5: Install PostgreSQL Client (Lightweight)

```bash
# Install PostgreSQL client tools only (not the server)
sudo dnf install -y postgresql15

# Verify
psql --version
```

**Why:** Allows CLI access to PostgreSQL without running server locally.

---

#### Step 6: Install kubectl and AWS Tools

```bash
# Install kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install AWS CLI v2 (for AWS resource management)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/
aws --version

# Install aws-iam-authenticator (for EKS cluster access)
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.27.0/2023-07-04/bin/linux/amd64/aws-iam-authenticator
sudo install -o root -g root -m 0755 aws-iam-authenticator /usr/local/bin/aws-iam-authenticator
```

**Best Practices:**
- Using official binaries from trusted sources
- Installing to /usr/local/bin for system-wide access
- Modular tool installation (only required tools)

---

#### Step 7: Install Terraform (Infrastructure as Code)

```bash
# Download latest Terraform
TERRAFORM_VERSION="1.6.0"
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo dnf update -y
sudo dnf install -y terraform

# Verify
terraform version
```

---

#### Step 8: Install Git and Configure SSH Keys

```bash
# Git is likely already installed, verify
git --version

# Generate SSH key for GitHub (if not already done)
ssh-keygen -t ed25519 -C "your-email@example.com" -N "" -f ~/.ssh/id_ed25519

# Display public key (add to GitHub account)
cat ~/.ssh/id_ed25519.pub

# Test GitHub connection
ssh -T git@github.com
```

**Best Practices:**
- Using Ed25519 (more secure than RSA)
- No passphrase (for CI/CD automation)

---

#### Step 9: Install Jenkins (Optional - if acting as CI/CD Server)

```bash
# Install Java (required for Jenkins)
sudo dnf install -y java-17-amazon-corretto-devel

# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Jenkins
sudo dnf install -y jenkins

# Start and enable Jenkins
sudo systemctl daemon-reload
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Access Jenkins at http://your-instance-ip:8080
```

---

#### Step 10: Install Helm (Package Manager for Kubernetes)

```bash
# Download Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

**Why:** Simplifies Kubernetes application deployment.

---

#### Step 11: System Hardening (Security Best Practice)

```bash
# Enable firewall (if not using AWS security groups)
sudo systemctl enable firewalld
sudo systemctl start firewalld

# Allow required ports
sudo firewall-cmd --permanent --add-port=22/tcp     # SSH
sudo firewall-cmd --permanent --add-port=8080/tcp   # Jenkins (if applicable)
sudo firewall-cmd --permanent --add-port=80/tcp     # HTTP
sudo firewall-cmd --permanent --add-port=443/tcp    # HTTPS
sudo firewall-cmd --reload

# SELinux configuration (permissive for Docker)
sudo semanage fcontext -a -t container_file_t "/var/lib/docker" 2>/dev/null || true

# Set up log rotation for Docker
sudo tee /etc/logrotate.d/docker-container > /dev/null <<EOF
/var/lib/docker/containers/*/*.log {
  rotate 7
  daily
  compress
  missingok
  delaycompress
  copytruncate
}
EOF
```

---

#### Step 12: Create Non-Root User for Application (Security Best Practice)

```bash
# Create dedicated user for task manager app
sudo useradd -m -s /bin/bash taskmanager
sudo usermod -aG docker taskmanager

# Create application directory
sudo mkdir -p /opt/taskmanager
sudo chown -R taskmanager:taskmanager /opt/taskmanager

# Switch to new user for application setup
sudo -u taskmanager -s
cd /opt/taskmanager
```

**Best Practices:**
- Dedicated user limits security blast radius
- Application runs as non-root user
- Group permissions for Docker access

---

### Verification Commands

```bash
# Verify all installations
echo "=== System Information ==="
uname -a
lsb_release -a

echo "=== Docker Setup ==="
docker --version
docker run hello-world

echo "=== Python Setup ==="
python3 --version
python3 -m venv test_venv && source test_venv/bin/activate && pip --version

echo "=== Kubernetes Tools ==="
kubectl version --client
helm version

echo "=== AWS Tools ==="
aws --version
aws-iam-authenticator version

echo "=== Infrastructure Tools ==="
terraform version
git --version

echo "=== Database Client ==="
psql --version
```

---

## 🐳 Local Deployment with Docker Compose

### Quick Start

```bash
# Clone the repository
git clone <your-repo-url>
cd Python-DevOps

# Create environment file
cat > .env.local <<EOF
FLASK_ENV=development
SECRET_KEY=7a8f9d2e4c6b1a3e5d7f9b2c4e6a8d0f1c3e5a7b9d1f3e5c7a9b1d3f5e7a9c1b
DATABASE_URL=postgresql://taskmanager:P@ssw0rd!2026SecureDB@postgres:5432/taskmanager_db
EOF

# Build and start containers
docker-compose up --build

# Access application
# Frontend:  http://localhost
# Backend:   http://localhost:8888
# Postgres:  localhost:5432
```

### Docker Compose Workflow Details

```bash
# View running containers
docker-compose ps

# View logs in real-time
docker-compose logs -f

# Stop all containers
docker-compose down

# Rebuild images after code changes
docker-compose up --build --f

# Run specific service
docker-compose up postgres  # Start only database

# Execute command in running container
docker-compose exec backend python run.py

# Remove all volumes (fresh start)
docker-compose down -v
```

---

## 🏗️ AWS Infrastructure Setup with Terraform

### Terraform Configuration Overview

```bash
# Initialize Terraform
cd terraform
terraform init

# Validate configuration
terraform validate

# Plan infrastructure (preview changes)
terraform plan -out=plan.tfplan

# Apply configuration (create resources)
terraform apply plan.tfplan

# View outputs
terraform output

# Destroy infrastructure (when done)
terraform destroy
```

### Key Resources Provisioned

| Resource | Purpose |
|----------|---------|
| **VPC** | Virtual network with public/private subnets |
| **EKS Cluster** | Kubernetes control plane |
| **EC2 Nodes** | Worker nodes for cluster |
| **RDS PostgreSQL** | Managed relational database |
| **ECR** | Container image registry |
| **ALB** | Application load balancer |
| **Security Groups** | Network access control |
| **IAM Roles** | AWS permissions management |

---

## ⚙️ Jenkins CI/CD Pipeline Setup

### Pipeline Stages Explained

**Stage 1: Git Checkout**
- Pulls latest code from repository
- Ensures CI starts with current source

**Stage 2: SonarQube Analysis**
- Scans code for bugs, vulnerabilities, code smells
- Quality gate blocks deployment if standards not met
- Enforces code quality baseline

**Stage 3: Docker Build**
- Creates container images for backend and frontend
- Tags with version and build number
- Ensures reproducible builds

**Stage 4: Container Testing**
- Spins up application using Docker Compose
- Performs HTTP health checks
- Validates connectivity between services
- Ensures buildable image runs correctly

**Stage 5: Push to Registry**
- Authenticates to Docker Hub / ECR
- Uploads built images
- Makes images available for deployment

**Stage 6: Kubernetes Deployment**
- Creates namespace isolation
- Applies secrets securely
- Deploys PostgreSQL with persistent storage
- Deploys backend and frontend services
- Configures ingress routing
- Waits for successful rollout

**Stage 7: Monitoring Setup**
- Deploys Prometheus for metrics collection
- Deploys Grafana for visualization
- Configures dashboards and alerts

---

## 🚀 Kubernetes Deployment

### Deploy to EKS Cluster

```bash
# Configure kubectl to access EKS cluster
aws eks update-kubeconfig --name taskmanager-cluster --region us-east-1

# Verify cluster access
kubectl cluster-info
kubectl get nodes

# Create namespace
kubectl create namespace taskmanager

# Apply secrets
kubectl apply -f k8s/secrets.yaml -n taskmanager

# Deploy database first
kubectl apply -f k8s/postgres-pvc.yaml -n taskmanager
kubectl apply -f k8s/postgres-deployment.yaml -n taskmanager
kubectl wait --for=condition=available --timeout=300s deployment/postgres -n taskmanager

# Deploy backend
kubectl apply -f k8s/backend-deployment.yaml -n taskmanager
kubectl wait --for=condition=available --timeout=300s deployment/backend -n taskmanager

# Deploy frontend
kubectl apply -f k8s/frontend-deployment.yaml -n taskmanager
kubectl wait --for=condition=available --timeout=300s deployment/frontend -n taskmanager

# Setup ingress
kubectl apply -f k8s/ingress.yaml -n taskmanager

# Get ingress URL
kubectl get ingress -n taskmanager

# Verify deployments
kubectl get all -n taskmanager
```

### Health Checking and Troubleshooting

```bash
# Check pod status
kubectl get pods -n taskmanager -o wide

# View pod logs
kubectl logs -f deployment/backend -n taskmanager
kubectl logs -f deployment/frontend -n taskmanager

# Describe pod (detailed information)
kubectl describe pod <pod-name> -n taskmanager

# Check resource usage
kubectl top nodes
kubectl top pods -n taskmanager

# Check persistent volume claims
kubectl get pvc -n taskmanager
```

---

## 📊 Monitoring & Observability

### Deploy Monitoring Stack

```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Deploy Prometheus
kubectl apply -f monitoring/prometheus-rbac.yaml
kubectl apply -f monitoring/prometheus-config.yaml
kubectl apply -f monitoring/prometheus-deployment.yaml

# Deploy Grafana
kubectl apply -f monitoring/grafana-datasource.yaml
kubectl apply -f monitoring/grafana-dashboard-config.yaml
kubectl apply -f monitoring/grafana-deployment.yaml

# Access Grafana
kubectl port-forward svc/grafana 3000:80 -n monitoring
# Open http://localhost:3000 (admin / admin)
```

### Key Metrics to Monitor

```
Application Metrics:
• HTTP request latency
• Request success/failure rate
• Database query latency
• Backend memory/CPU usage

Infrastructure Metrics:
• Node CPU utilization
• Node memory usage
• Disk space available
• Network I/O
• Pod restart count
```

---

## 🐛 Troubleshooting Guide

### Common Issues and Solutions

#### Issue: Container fails to start

```bash
# Check logs
docker logs <container-id>

# Inspect image
docker inspect <image-id>

# Run with verbose output
docker run -it --rm <image> /bin/bash
```

#### Issue: PostgreSQL connection refused

```bash
# Verify database is running
kubectl get pod -l app=postgres -n taskmanager

# Check database logs
kubectl logs -f deployment/postgres -n taskmanager

# Verify connection string
kubectl get secret -n taskmanager -o yaml
```

#### Issue: Ingress not routing traffic

```bash
# Check ingress status
kubectl describe ingress taskmanager-ingress -n taskmanager

# Check service endpoints
kubectl get endpoints -n taskmanager

# Test DNS resolution
kubectl run -it --rm debug --image=alpine --restart=Never -- nslookup kubernetes.default
```

#### Issue: Pods stuck in pending state

```bash
# Check node capacity
kubectl describe nodes

# Check pod resource requests
kubectl describe pod <pod-name> -n taskmanager

# Check for resource quotas or limits
kubectl get resourcequota -A
```

---

## 🔒 Security Best Practices

### Kubernetes Security

✅ **Network Policies**
- Restrict pod-to-pod communication
- Whitelist necessary traffic only

✅ **RBAC (Role-Based Access Control)**
- Least privilege access model
- Service accounts with minimal permissions
- No* `cluster-admin` for applications

✅ **Secrets Management**
- Never commit secrets to Git
- Use Kubernetes Secrets or AWS Secrets Manager
- Encrypt secrets at rest
- Rotate secrets regularly

✅ **Pod Security**
- Non-root user execution
- Read-only root filesystem
- Resource limits and requests
- Health checks (liveness, readiness probes)

✅ **Container Image Security**
- Scan images for vulnerabilities
- Use minimal base images (Alpine, Distroless)
- Pin image versions (no `latest` tags)
- Sign images with container signatures

### AWS Security

✅ **IAM Policies**
- Principle of least privilege
- Regular access reviews
- MFA for production access

✅ **Network Security**
- VPC with public/private subnets
- Security groups with minimal rules
- NACLs for additional network control
- VPN/bastion hosts for cluster access

✅ **Encryption**
- TLS for data in transit
- EBS encryption for data at rest
- KMS for key management

### Docker Security

✅ **Base Image Updates**
- Regularly rebuild images
- Use security-scanned base images
- Implement image signing

✅ **Runtime Security**
- Non-root containers
- Read-only filesystems where possible
- Resource limits
- No privileged mode

✅ **Supply Chain**
- Signed commits and tags
- Vulnerability scanning in CI/CD
- SBOM (Software Bill of Materials)

---

## 🤝 Contributing

### Development Workflow

```bash
# 1. Create feature branch
git checkout -b feature/your-feature

# 2. Make changes
# Edit files, test locally

# 3. Build and test locally
docker-compose up --build
# Run tests
python3 -m pytest backend/tests/

# 4. Commit and push
git add .
git commit -m "feat: description of changes"
git push origin feature/your-feature

# 5. Create Pull Request
# Jenkins CI pipeline runs automatically
# Must pass all checks before merge
```

### Code Quality Standards

- Python code follows PEP 8
- JavaScript follows ESLint standards
- SQL follows best practices
- Kubernetes YAML is validated with kubeval
- Terraform follows best practices

---

## 📚 Additional Resources

### Project Documentation
- [INTERVIEW-PROJECT-EXPLANATION.md](./INTERVIEW-PROJECT-EXPLANATION.md) - Detailed interview explanation
- [terraform/README.md](./terraform/README.md) - Infrastructure as Code documentation

### External Resources
- [Flask Documentation](https://flask.palletsprojects.com/)
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [AWS EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [SonarQube Documentation](https://docs.sonarqube.org/)

---

## 📄 License

This project is provided as-is for educational and portfolio purposes.

---

## ✉️ Support

For issues or questions:
1. Check [Troubleshooting Guide](#troubleshooting-guide)
2. Review logs (Docker, Kubernetes, Jenkins)
3. Verify prerequisites are installed correctly
4. Check AWS resource quotas and limits

---

**Last Updated:** April 2026
**Version:** 1.0.0
**Maintainer:** DevOps Team

## ⚙️ Deployment Options

### Full Deployment (Terraform + Jenkins)
- Provision infrastructure using Terraform  
- Configure Jenkins  
- Run CI/CD pipeline  
- Deploy automatically to EKS  

### Local Deployment (Docker)

```bash
docker compose up -d --build
````

### Manual Kubernetes Deployment

```bash
kubectl apply -f k8s/
```

---

## 📊 Monitoring

* Prometheus for metrics collection
* Grafana for visualization
* Integrated with Kubernetes cluster

---

## 📌 Key Highlights

* Automated CI/CD pipeline
* Infrastructure as Code using Terraform
* Containerized microservices architecture
* Kubernetes-based deployment
* Monitoring and observability setup

---

## 🚀 Future Improvements

* Add autoscaling (HPA)
* Improve security (IAM roles, secrets management)
* Add HTTPS with domain
* Implement blue-green deployment

---

## 📘 Detailed Setup Guide

<details>
<summary>Click to expand</summary>

### Terraform Setup

```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

### EC2 Setup

```bash
sudo dnf -y update
sudo dnf -y install git docker
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
```

### AWS CLI

```bash
aws configure
```

### Install Kubernetes Tools

```bash
curl -LO https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

curl -sL https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz | tar -xz
sudo mv eksctl /usr/local/bin/
```

### Create EKS Cluster

```bash
eksctl create cluster --name taskmanager-eks --region us-east-1 --nodes 2
aws eks update-kubeconfig --name taskmanager-eks --region us-east-1
kubectl get nodes
```

### Jenkins Setup

```bash
sudo dnf install java-17-amazon-corretto-devel -y
sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Access:

```
http://<EC2_PUBLIC_IP>:8080
```

### Pipeline

* Connect GitHub repo
* Configure using Jenkinsfile
* Run build

### Monitoring

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

### Verify

```bash
kubectl get pods
kubectl get svc
```

</details>

---

## 👨‍💻 Author

Saikiran Asamwar
GitHub: [https://github.com/SaikiranAsamwar](https://github.com/SaikiranAsamwar)
DockerHub: [https://hub.docker.com/u/saikiranasamwar4](https://hub.docker.com/u/saikiranasamwar4)

```
```

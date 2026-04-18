```markdown
# 🚀 Cloud-Native Task Manager – DevOps Project

A cloud-native Task Manager application built using **Flask + PostgreSQL** and deployed on **AWS EKS** using a complete **CI/CD pipeline**, **Docker containerization**, and **Infrastructure as Code (Terraform)**.

---

## 📌 Project Overview

- CI/CD pipeline using Jenkins  
- Docker-based containerization  
- Kubernetes deployment on AWS EKS  
- Infrastructure provisioning using Terraform  
- Monitoring with Prometheus and Grafana  

---

## 🏗️ Architecture

```

User → Internet → AWS EKS (Kubernetes)
├── Frontend (Nginx)
├── Backend (Flask API)
└── Database (PostgreSQL)

GitHub → Jenkins → Docker → DockerHub → EKS Deployment

Monitoring → Prometheus + Grafana

```

---

## 📸 Project Screenshots

### 🔹 Architecture Diagram
![Architecture](./assets/architecture.png)

### 🔹 CI/CD Pipeline (Jenkins)
![Jenkins Pipeline](./assets/jenkins.png)

### 🔹 Application Running
![App](./assets/app.png)

### 🔹 Kubernetes Deployment
![K8s](./assets/k8s.png)

---

## 🛠️ Tech Stack

| Layer | Tools |
|------|------|
| Cloud | AWS (EKS, EC2, EBS, ELB) |
| CI/CD | Jenkins |
| Containerization | Docker |
| Orchestration | Kubernetes |
| IaC | Terraform |
| Monitoring | Prometheus, Grafana |
| Code Quality | SonarQube |
| Backend | Python Flask |
| Database | PostgreSQL |

---

## 🔄 CI/CD Pipeline

```

GitHub → Jenkins → Build → Test → Docker Image → Push → Deploy to EKS

```

- Source Code Checkout  
- Code Quality Analysis (SonarQube)  
- Docker Image Build  
- Container Testing  
- Push to DockerHub  
- Deployment to EKS  
- Monitoring Setup  

---

## 📂 Project Structure

```

backend/
frontend/
k8s/
terraform/
jenkins/
monitoring/
docker-compose.yml
Jenkinsfile

````

---

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

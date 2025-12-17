# 🐳 Task Manager - Complete Deployment Guide

Full-stack task management application with Flask backend, Nginx frontend, and PostgreSQL database. This guide covers local development, AWS EC2 deployment, Kubernetes/EKS deployment, and Jenkins CI/CD pipeline setup.

**Stack:** Nginx + Flask + PostgreSQL | **Orchestration:** Docker Compose, Kubernetes | **CI/CD:** Jenkins

---

## 📑 Table of Contents

- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Local Deployment](#-local-deployment)
- [AWS EC2 Deployment](#-aws-ec2-deployment)
- [Kubernetes/EKS Deployment](#-kuberneteseks-deployment)
- [Jenkins CI/CD Setup](#-jenkins-cicd-setup)
- [Monitoring & Logging](#-monitoring--logging)
- [Troubleshooting](#-troubleshooting)
- [Destroy Deployment](#-destroy-deployment)

---

## 📌 Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Frontend** | Nginx (Alpine) | Latest |
| **Backend** | Python Flask | 3.11 |
| **Database** | PostgreSQL | 15-alpine |
| **Container Runtime** | Docker | 20.10+ |
| **Container Orchestration** | Docker Compose / Kubernetes | v2 / v1.28+ |
| **Container Registry** | Docker Hub | saikiranasamwar4/* |
| **CI/CD** | Jenkins | 2.4+ |
| **Cloud Platform** | AWS (EC2, EKS, ALB) | - |

---

## 🔧 Prerequisites

### Local Development

- **Docker Desktop** (20.10+) - [Download](https://www.docker.com/products/docker-desktop)
- **Docker Compose** (v2+) - Included with Docker Desktop
- **Git** - [Download](https://git-scm.com/)
- **4GB RAM minimum** (8GB recommended)
- **10GB free disk space**

### AWS Deployment

#### EC2 Requirements
- AWS Account with EC2 access
- Key pair (.pem file) for SSH access
- Security group with ports: 22, 80, 443, 5000
- Amazon Linux 2023 instance (t2.medium or higher)

#### EKS Requirements
- AWS CLI v2 configured with credentials
- kubectl v1.28+ - [Install](https://kubernetes.io/docs/tasks/tools/)
- eksctl - [Install](https://eksctl.io/installation/)
- IAM permissions for EKS cluster creation
- VPC with public/private subnets

### Jenkins CI/CD Requirements
- Jenkins server (2.4+) with Docker plugin
- Docker Hub account and credentials
- Kubernetes cluster access (kubeconfig file)
- Jenkins plugins:
  - Docker Pipeline
  - Kubernetes CLI
  - Git
  - Credentials Binding

---

## 🚀 Local Deployment

### Quick Start (Docker Compose)

```bash
# Clone repository
git clone https://github.com/SaikiranAsamwar/Python-DevOps.git
cd Python-DevOps

# Start all services
docker-compose up -d --build

# Verify deployment
docker-compose ps
```

**Expected Output:**
```
NAME                COMMAND                  SERVICE             STATUS              PORTS
backend             "python run.py"          backend             running             0.0.0.0:5000->5000/tcp
db                  "docker-entrypoint.s…"   db                  running             5432/tcp
frontend            "/docker-entrypoint.…"   frontend            running             0.0.0.0:80->80/tcp
```

**Access Application:**
- **Frontend UI**: http://localhost
- **Backend API**: http://localhost:5000
- **API Health Check**: http://localhost:5000/health

### Verify Services

```bash
# Check container logs
docker-compose logs -f

# Check specific service
docker-compose logs backend

# View real-time resource usage
docker stats

# Test database connection
docker exec -it db psql -U devops_user -d devops_db
```

---

## ☁️ AWS EC2 Deployment

### Step 1: Launch EC2 Instance

1. **Create EC2 Instance:**
   - AMI: Amazon Linux 2023
   - Instance Type: t2.medium (2 vCPU, 4GB RAM minimum)
   - Storage: 20GB EBS volume
   - Key pair: Create or use existing .pem file

2. **Configure Security Group:**
   ```
   Inbound Rules:
   - SSH (22): Your IP
   - HTTP (80): 0.0.0.0/0
   - HTTPS (443): 0.0.0.0/0
   - Custom TCP (5000): 0.0.0.0/0 (for API)
   ```

### Step 2: Connect to EC2

```bash
# Windows (PowerShell)
ssh -i "C:\path\to\your-key.pem" ec2-user@<EC2_PUBLIC_IP>

# Linux/Mac
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>
```

### Step 3: Install Docker & Docker Compose

```bash
# Update system packages
sudo yum update -y

# Install Docker
sudo yum install git docker -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add ec2-user to docker group (no sudo needed)
sudo usermod -aG docker ec2-user

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installations
docker --version
docker-compose --version

# Log out and back in for group changes
exit
```

### Step 4: Deploy Application

```bash
# Reconnect to EC2
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>

# Clone repository
git clone https://github.com/SaikiranAsamwar/Python-DevOps.git
cd Python-DevOps

# Start application
docker-compose up -d --build

# Verify containers are running
docker-compose ps

# Check logs
docker-compose logs -f
```

### Step 5: Access Application

**Public URLs:**
- **Frontend**: http://\<EC2_PUBLIC_IP\>
- **Backend API**: http://\<EC2_PUBLIC_IP\>:5000
- **Health Check**: http://\<EC2_PUBLIC_IP\>:5000/health

**Get EC2 Public IP:**
```bash
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

> **Note:** For production deployments, configure SSL/TLS certificates and use HTTPS. See [DOCKER_DEPLOYMENT_AWS.md](DOCKER_DEPLOYMENT_AWS.md) for detailed configuration.

---

## ☸️ Kubernetes/EKS Deployment

### Prerequisites Check

```bash
# Verify AWS CLI
aws --version

# Verify kubectl
kubectl version --client

# Verify eksctl
eksctl version

# Configure AWS credentials
aws configure
```

### Step 1: Create EKS Cluster

```bash
# Create cluster (takes ~15-20 minutes)
eksctl create cluster \
  --name task-manager-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Verify cluster
kubectl get nodes
```

**Expected Output:**
```
NAME                             STATUS   ROLES    AGE   VERSION
ip-192-168-x-x.ec2.internal      Ready    <none>   5m    v1.28.x
ip-192-168-y-y.ec2.internal      Ready    <none>   5m    v1.28.x
```

### Step 2: Deploy Application

```bash
# Clone repository
git clone https://github.com/SaikiranAsamwar/Python-DevOps.git
cd Python-DevOps

# Apply Kubernetes manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/postgres-pvc.yaml
kubectl apply -f k8s/postgres-deployment.yaml

# Wait for database to be ready
kubectl wait --for=condition=ready pod -l app=postgres -n task-manager --timeout=300s

# Deploy backend and frontend
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml

# Optional: Deploy ingress (ALB)
kubectl apply -f k8s/ingress.yaml

# Verify deployments
kubectl get all -n task-manager
```

### Step 3: Access Application

```bash
# Get LoadBalancer URL (wait 2-3 minutes for provisioning)
kubectl get svc frontend-service -n task-manager

# Example output:
# NAME               TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)
# frontend-service   LoadBalancer   10.100.200.50   a1b2c3d4e5f6g7h8-123456789.us-east-1.elb.amazonaws.com                 80:30080/TCP
```

**Access URLs:**
- **Frontend**: http://\<EXTERNAL-IP\>
- **Backend API**: http://\<EXTERNAL-IP\>/api

### Step 4: Monitor Deployment

```bash
# Check pod status
kubectl get pods -n task-manager

# View pod logs
kubectl logs -f deployment/backend -n task-manager
kubectl logs -f deployment/frontend -n task-manager

# Check rollout status
kubectl rollout status deployment/backend -n task-manager
kubectl rollout status deployment/frontend -n task-manager

# Describe pods for troubleshooting
kubectl describe pods -n task-manager
```

### Scaling

```bash
# Scale backend
kubectl scale deployment/backend --replicas=3 -n task-manager

# Scale frontend
kubectl scale deployment/frontend --replicas=3 -n task-manager

# Enable autoscaling
kubectl autoscale deployment backend --cpu-percent=70 --min=2 --max=10 -n task-manager
```

---

## 🔄 Jenkins CI/CD Setup

### Step 1: Install Jenkins

**Using Docker:**
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

**Access Jenkins:**
- URL: http://localhost:8080
- Get initial password: `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`

### Step 2: Install Required Plugins

Navigate to **Manage Jenkins** → **Manage Plugins** → **Available**

Install:
- Docker Pipeline
- Kubernetes CLI Plugin
- Git Plugin
- Credentials Binding Plugin
- Pipeline

### Step 3: Configure Credentials

**Manage Jenkins** → **Credentials** → **Global** → **Add Credentials**

1. **Docker Hub Credentials**
   - Kind: Username with password
   - ID: `dockerhub-credentials`
   - Username: Your Docker Hub username
   - Password: Docker Hub access token

2. **Kubernetes Config**
   - Kind: Secret file
   - ID: `kubeconfig-eks`
   - File: Upload your `~/.kube/config` file

### Step 4: Create Pipeline Job

1. **New Item** → **Pipeline** → Name: "task-manager-pipeline"
2. **Pipeline** section:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: `https://github.com/SaikiranAsamwar/Python-DevOps.git`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`
3. **Save** and **Build Now**

### Pipeline Stages

The Jenkins pipeline automates:

1. **Checkout** - Clone repository from Git
2. **Build Backend** - Create Docker image for Flask backend
3. **Build Frontend** - Create Docker image for Nginx frontend
4. **Push to Docker Hub** - Upload images with build number tag
5. **Deploy to Kubernetes** - Update EKS cluster with new images

### Monitor Pipeline

```bash
# View pipeline console output in Jenkins UI
# Check build history and stage view

# Verify deployment in Kubernetes
kubectl get pods -n task-manager
kubectl get deployments -n task-manager

# Check deployed image tags
kubectl describe deployment backend -n task-manager | grep Image
kubectl describe deployment frontend -n task-manager | grep Image
```

---

## 📊 Monitoring & Logging

### Docker Compose Monitoring

```bash
# View all service logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f db

# Check container status
docker-compose ps

# View real-time resource usage
docker stats

# Monitor container health
docker inspect --format='{{.State.Health.Status}}' backend
```

### Kubernetes Monitoring

```bash
# View pod logs
kubectl logs -f deployment/backend -n task-manager
kubectl logs -f deployment/frontend -n task-manager
kubectl logs -f statefulset/postgres -n task-manager

# Get pod metrics
kubectl top pods -n task-manager
kubectl top nodes

# Check events
kubectl get events -n task-manager --sort-by='.lastTimestamp'

# View pod status
kubectl get pods -n task-manager -o wide

# Check service endpoints
kubectl get endpoints -n task-manager
```

### Application Health Checks

```bash
# Backend health (Docker Compose)
curl http://localhost:5000/health

# Backend health (Kubernetes)
kubectl run curl --image=curlimages/curl -i --rm --restart=Never -n task-manager -- \
  curl http://backend-service:5000/health

# Check database connectivity
docker exec -it db psql -U devops_user -d devops_db -c "SELECT version();"

# Kubernetes database check
kubectl exec -it statefulset/postgres -n task-manager -- \
  psql -U devops_user -d devops_db -c "SELECT version();"
```

---

## 🚨 Troubleshooting

### Docker Compose Issues

#### Error: Port already in use

**Symptoms:** `Bind for 0.0.0.0:80 failed: port is already allocated`

```bash
# Windows: Find and kill process
netstat -ano | findstr :80
taskkill /PID <PID> /F

# Linux/Mac: Find and kill process
sudo lsof -i :80
sudo kill -9 <PID>

# Alternative: Change port in docker-compose.yml
# frontend:
#   ports:
#     - "8080:80"  # Use port 8080 instead
```

#### Error: Database connection failed

**Symptoms:** `psycopg2.OperationalError: could not connect to server`

```bash
# Check database status
docker-compose ps db

# Restart database container
docker-compose restart db

# View database logs
docker-compose logs db

# Wait for database to be ready
docker-compose up -d db
sleep 10
docker-compose up -d backend
```

#### Error: Permission denied (Docker commands)

**Symptoms:** `Got permission denied while trying to connect to the Docker daemon socket`

```bash
# Linux: Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Or run with sudo
sudo docker-compose up -d
```

#### Error: Out of disk space

**Symptoms:** `no space left on device`

```bash
# Check disk usage
docker system df

# Clean up unused resources
docker system prune -a --volumes

# Remove specific components
docker image prune -a
docker volume prune
docker network prune

# Nuclear option (removes everything)
docker system prune -a --volumes -f
```

#### Error: Container fails to start

**Symptoms:** Container exits immediately after starting

```bash
# View container logs
docker-compose logs -f <service_name>

# Check container exit code
docker-compose ps

# Rebuild without cache
docker-compose build --no-cache
docker-compose up -d

# Run container interactively for debugging
docker-compose run backend /bin/bash
```

### Kubernetes/EKS Issues

#### Error: ImagePullBackOff

**Symptoms:** Pod stuck in `ImagePullBackOff` state

```bash
# Check pod events
kubectl describe pod <pod-name> -n task-manager

# Verify image exists in Docker Hub
docker pull saikiranasamwar4/task-manager-backend:latest

# Check image pull secrets
kubectl get secrets -n task-manager

# Manually pull image on node
kubectl debug node/<node-name> -it --image=busybox
```

#### Error: CrashLoopBackOff

**Symptoms:** Pod continuously restarting

```bash
# View pod logs
kubectl logs <pod-name> -n task-manager --previous

# Check pod events
kubectl describe pod <pod-name> -n task-manager

# Check resource limits
kubectl describe pod <pod-name> -n task-manager | grep -A 5 "Limits"

# Increase resources in deployment yaml
# resources:
#   limits:
#     memory: "1Gi"
#     cpu: "1000m"
```

#### Error: Pending Pods

**Symptoms:** Pods stuck in `Pending` state

```bash
# Check pod events
kubectl describe pod <pod-name> -n task-manager

# Check node resources
kubectl top nodes

# Check PVC status
kubectl get pvc -n task-manager

# Common causes:
# - Insufficient node resources
# - PVC not bound
# - Node selector mismatch
# - Taints and tolerations
```

#### Error: LoadBalancer Pending

**Symptoms:** `EXTERNAL-IP` shows `<pending>` for LoadBalancer service

```bash
# Check service
kubectl describe svc frontend-service -n task-manager

# Wait 2-3 minutes for AWS to provision ELB
kubectl get svc frontend-service -n task-manager --watch

# Verify AWS Load Balancer Controller is installed
kubectl get deployment -n kube-system aws-load-balancer-controller

# Check service annotations
kubectl get svc frontend-service -n task-manager -o yaml
```

#### Error: Database Connection Timeout in Kubernetes

**Symptoms:** Backend can't connect to PostgreSQL

```bash
# Check if postgres pod is running
kubectl get pods -n task-manager -l app=postgres

# Check service endpoints
kubectl get endpoints postgres-service -n task-manager

# Test connectivity from backend pod
kubectl exec -it deployment/backend -n task-manager -- \
  nc -zv postgres-service 5432

# Check database logs
kubectl logs -f statefulset/postgres -n task-manager

# Verify secrets are mounted
kubectl describe pod <backend-pod> -n task-manager | grep -A 10 Environment
```

### Jenkins Pipeline Issues

#### Error: Docker permission denied in Jenkins

**Solution:**
```bash
# Add jenkins user to docker group on Jenkins server
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

#### Error: kubectl command not found

**Solution:**
```bash
# Install kubectl in Jenkins container
docker exec -u root jenkins bash -c "curl -LO https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl && chmod +x kubectl && mv kubectl /usr/local/bin/"
```

#### Error: Invalid kubeconfig

**Solution:**
- Verify kubeconfig file is uploaded correctly in Jenkins credentials
- Check file permissions
- Ensure cluster endpoint is accessible from Jenkins server
- Test: `kubectl --kubeconfig=/path/to/config get nodes`

---

## 🗑️ Destroy Deployment

### Docker Compose Cleanup

```bash
# Stop and remove containers
docker-compose down

# Remove containers, volumes, and images
docker-compose down -v --rmi all

# Remove orphaned containers
docker-compose down -v --rmi all --remove-orphans

# Complete cleanup
docker system prune -a --volumes -f
```

### Kubernetes/EKS Cleanup

```bash
# Delete application resources
kubectl delete namespace task-manager

# Or delete individual resources
kubectl delete -f k8s/

# Delete EKS cluster (takes ~10 minutes)
eksctl delete cluster --name task-manager-cluster --region us-east-1

# Verify deletion
eksctl get cluster --region us-east-1
```

### AWS EC2 Cleanup

```bash
# On EC2 instance
docker-compose down -v --rmi all

# In AWS Console:
# 1. Terminate EC2 instance
# 2. Delete associated EBS volumes
# 3. Delete Security Group (if not default)
# 4. Delete Key Pair (if no longer needed)
```

### Jenkins Cleanup

```bash
# Remove Jenkins container
docker stop jenkins
docker rm jenkins

# Remove Jenkins volume (WARNING: deletes all Jenkins data)
docker volume rm jenkins_home
```

---

## � Security Best Practices

### Docker Compose

```bash
# Use environment variables for secrets
# Create .env file (add to .gitignore)
DB_USER=devops_user
DB_PASSWORD=secure_password_here
DB_NAME=devops_db

# Update docker-compose.yml to use env vars
# environment:
#   - POSTGRES_USER=${DB_USER}
#   - POSTGRES_PASSWORD=${DB_PASSWORD}
```

### Kubernetes

```bash
# Use Kubernetes Secrets (base64 encoded)
kubectl create secret generic db-credentials \
  --from-literal=username=devops_user \
  --from-literal=password=secure_password \
  -n task-manager

# Use AWS Secrets Manager (recommended for production)
# - Store secrets in AWS Secrets Manager
# - Use External Secrets Operator to sync to K8s
```

### Jenkins

- Store credentials in Jenkins Credentials Store (never in Jenkinsfile)
- Use credential binding for sensitive data
- Enable CSRF protection
- Configure RBAC for user access
- Regular security updates

### Network Security

```bash
# AWS Security Groups - Principle of Least Privilege
# - Allow only necessary ports
# - Restrict SSH to specific IPs
# - Use VPN for internal access

# Kubernetes Network Policies
kubectl apply -f network-policy.yaml  # Restrict pod-to-pod traffic
```

---

## 📈 Performance Optimization

### Docker

```bash
# Multi-stage builds (already implemented)
# Use smaller base images (alpine)
# Minimize layers
# Use .dockerignore file

# Example .dockerignore:
*.md
.git
.gitignore
__pycache__
*.pyc
node_modules
```

### Kubernetes

```yaml
# Resource requests and limits
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Horizontal Pod Autoscaling
kubectl autoscale deployment backend \
  --cpu-percent=70 \
  --min=2 \
  --max=10 \
  -n task-manager
```

### Database

```bash
# PostgreSQL tuning
# Increase shared_buffers
# Adjust work_mem
# Enable connection pooling
# Regular VACUUM operations
```

---

## 🧪 Testing

### Local Testing

```bash
# Test backend API
curl -X GET http://localhost:5000/health
curl -X GET http://localhost:5000/api/tasks

# Test database connection
docker exec -it db psql -U devops_user -d devops_db -c "\dt"

# Load testing with Apache Bench
ab -n 1000 -c 10 http://localhost:5000/health
```

### Kubernetes Testing

```bash
# Smoke tests
kubectl run test-curl --image=curlimages/curl -i --rm --restart=Never -n task-manager -- \
  curl http://backend-service:5000/health

# Rolling update test
kubectl set image deployment/backend backend=saikiranasamwar4/task-manager-backend:new-version -n task-manager
kubectl rollout status deployment/backend -n task-manager

# Rollback test
kubectl rollout undo deployment/backend -n task-manager
```

---

## 📊 Project Structure

```
Python-DevOps/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   └── routes.py
│   ├── config.py
│   ├── run.py
│   ├── utils.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── templates/
│   │   ├── index.html
│   │   ├── login.html
│   │   └── *.html
│   ├── js/
│   │   ├── app.js
│   │   └── *.js
│   ├── css/
│   │   └── style.css
│   ├── nginx.conf
│   ├── default.conf
│   └── Dockerfile
├── k8s/
│   ├── namespace.yaml
│   ├── secrets.yaml
│   ├── postgres-pvc.yaml
│   ├── postgres-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── ingress.yaml
│   └── README.md
├── docker-compose.yml
├── Jenkinsfile
├── README.md
└── DOCKER_DEPLOYMENT_AWS.md
```

---

## 🎯 Quick Reference

### Essential Commands

| Task | Docker Compose | Kubernetes |
|------|---------------|-----------|
| **Deploy** | `docker-compose up -d` | `kubectl apply -f k8s/` |
| **View Logs** | `docker-compose logs -f` | `kubectl logs -f deployment/backend -n task-manager` |
| **Scale** | `docker-compose up -d --scale backend=3` | `kubectl scale deployment/backend --replicas=3 -n task-manager` |
| **Stop** | `docker-compose down` | `kubectl delete namespace task-manager` |
| **Status** | `docker-compose ps` | `kubectl get all -n task-manager` |
| **Restart** | `docker-compose restart` | `kubectl rollout restart deployment/backend -n task-manager` |

### Port Mapping

| Service | Local | EC2 | Kubernetes |
|---------|-------|-----|-----------|
| **Frontend** | :80 | :80 | LoadBalancer :80 |
| **Backend** | :5000 | :5000 | ClusterIP :5000 |
| **Database** | 5432 (internal) | 5432 (internal) | ClusterIP :5432 |

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is open source and available under the MIT License.

---

## 👨‍💻 Author

**Saikiran Rajesh Asamwar**

- **GitHub**: [@SaikiranAsamwar](https://github.com/SaikiranAsamwar)
- **Docker Hub**: [saikiranasamwar4](https://hub.docker.com/u/saikiranasamwar4)
- **Project Repository**: [EC2-Python-Docker](https://github.com/SaikiranAsamwar/EC2-Python-Docker)

---

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [AWS EKS Guide](https://docs.aws.amazon.com/eks/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [AWS EC2 Deployment Guide](DOCKER_DEPLOYMENT_AWS.md)
- [Kubernetes Manifests Guide](k8s/README.md)

---

## 📝 Changelog

### [2.0.0] - December 2025
- ➕ Added Kubernetes/EKS deployment support
- ➕ Added Jenkins CI/CD pipeline
- ➕ Added comprehensive monitoring and logging
- ➕ Added troubleshooting guide
- ➕ Added security best practices
- 📝 Updated README with complete deployment guide

### [1.0.0] - Initial Release
- ✨ Docker Compose deployment
- ✨ AWS EC2 deployment
- ✨ Multi-container orchestration

---

**Last Updated**: December 17, 2025

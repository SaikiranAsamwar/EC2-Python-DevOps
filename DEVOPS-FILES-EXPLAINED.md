# 📚 DevOps Infrastructure Files Explained

This document provides a **line-by-line breakdown** of all infrastructure, deployment, and DevOps configuration files in this project. This is meant for learning and interview preparation.

---

## Table of Contents

1. [Docker Files](#docker-files)
2. [Docker Compose](#docker-compose)
3. [Kubernetes Manifests](#kubernetes-manifests)
4. [Jenkins CI/CD Pipeline](#jenkins-cicd-pipeline)
5. [Terraform Infrastructure](#terraform-infrastructure)
6. [Monitoring & Observability](#monitoring--observability)
7. [Configuration Files](#configuration-files)

---

## Docker Files

### 1. Backend Dockerfile (`backend/Dockerfile`)

```dockerfile
FROM python:3.11-slim
```
- **Purpose**: Define the base image for the backend container
- **Why slim?**: Reduced image size (~170MB instead of ~900MB), faster pull/push, less attack surface
- **Python 3.11**: Latest stable version with latest features and security patches

```dockerfile
WORKDIR /app
```
- **Purpose**: Set working directory inside container
- **Effect**: All subsequent commands execute from `/app` directory
- **Best Practice**: Keeps container filesystem organized

```dockerfile
COPY requirements.txt .
```
- **Purpose**: Copy Python dependencies list from host to container
- **Why first?**: Enables Docker layer caching - if code changes but requirements don't, Docker reuses cached layer

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
- **Purpose**: Install all Python dependencies listed in requirements.txt
- **`--no-cache-dir`**: Prevents pip from storing cache files → reduces image size by ~100MB
- **Best Practice**: Always use this flag in Docker to minimize layer size

```dockerfile
COPY app app
COPY templates templates
COPY static static
COPY run.py .
```
- **Purpose**: Copy application code into container
- **Why after pip install?**: Source code changes frequently, but dependencies rarely do
- **Layer Strategy**: By copying dependencies first, rebuilds are faster when only code changes

```dockerfile
RUN apt-get update && apt-get install -y netcat-openbsd
```
- **Purpose**: Install `netcat` utility (network tool for testing connectivity)
- **Why needed?**: Used by `wait-for-db.sh` script to check if PostgreSQL is ready before starting app
- **Best Practice**: Combined `apt-get update` and `apt-get install` on same line to reduce layers

```dockerfile
COPY wait-for-db.sh .
RUN chmod +x wait-for-db.sh
```
- **Purpose**: Copy database waiting script and make it executable
- **`chmod +x`**: Makes script runnable (adds execute permission)
- **Why?**: Ensures app doesn't crash if database isn't ready yet

```dockerfile
EXPOSE 8888
```
- **Purpose**: Documents which port the container listens on (no security implications)
- **Value**: Port 8888 is where Flask app listens

```dockerfile
CMD ["sh", "wait-for-db.sh"]
```
- **Purpose**: Default command when container starts
- **Process**: Runs `wait-for-db.sh` which waits for PostgreSQL, then starts Flask app
- **Best Practice**: Using shell wrapper allows checking dependencies before starting app

---

### 2. Frontend Dockerfile (`frontend/Dockerfile`)

```dockerfile
FROM nginx:alpine AS builder
```
- **Multi-stage build**: First stage is named "builder"
- **nginx:alpine**: Lightweight Nginx image (~24MB instead of ~150MB)
- **Purpose**: This stage prepares frontend assets

```dockerfile
WORKDIR /build
COPY . .
```
- **Purpose**: Copy all frontend files (HTML, CSS, JS) into build context

```dockerfile
FROM nginx:alpine
```
- **Multi-stage build**: Second stage - final production image
- **Benefit**: Only final stage is kept; builder stage is discarded
- **Result**: Smaller image (no build tools/temporary files)

```dockerfile
COPY --from=builder /build /usr/share/nginx/html
```
- **Purpose**: Copy built assets from builder stage into Nginx serving directory
- **`/usr/share/nginx/html`**: Default Nginx document root
- **Benefit**: Only copies necessary files into final image

```dockerfile
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
- **Purpose**: Copy custom Nginx configuration
- **Location**: `/etc/nginx/conf.d/default.conf` is Nginx's default server config

```dockerfile
EXPOSE 80
```
- **Purpose**: Declare container listens on HTTP port 80

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```
- **Purpose**: Start Nginx server
- **`daemon off;`**: Run Nginx in foreground (required for Docker, allows container to stay running)

---

## Docker Compose

### `docker-compose.yml`

**Purpose**: Define multi-container application for local development/testing

```yaml
services:
  postgres:
    image: postgres:15-alpine
```
- **Service name**: `postgres` - containers can reference this by hostname
- **Image**: PostgreSQL 15 (latest stable) on Alpine Linux (lightweight)

```yaml
    environment:
      POSTGRES_USER: taskmanager
      POSTGRES_PASSWORD: "P@ssw0rd!2026SecureDB"
      POSTGRES_DB: taskmanager_db
```
- **POSTGRES_USER**: Admin username for database
- **POSTGRES_PASSWORD**: Database password (WARNING: hardcoded - only for dev!)
- **POSTGRES_DB**: Default database created on startup

```yaml
    volumes:
      - postgres_data:/var/lib/postgresql/data
```
- **Purpose**: Persist database data across container restarts
- **Named volume**: Docker manages `/var/lib/postgresql/data` storage
- **Benefit**: Stop container, restart it, data is still there

```yaml
    ports:
      - "5432:5432"
```
- **Format**: `HOST_PORT:CONTAINER_PORT`
- **Meaning**: Forward port 5432 on your machine to container's port 5432
- **Access**: Can connect to database from host via `localhost:5432`

```yaml
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U taskmanager"]
      interval: 5s
      timeout: 5s
      retries: 5
```
- **Purpose**: Periodically check if PostgreSQL is ready
- **`pg_isready`**: Utility that returns 0 if database is accepting connections
- **Logic**: Every 5s, run health check; if OK, mark as healthy; if fails 5 times, mark as unhealthy
- **Benefit**: Docker won't start backend until PostgreSQL is healthy

```yaml
  backend:
    build: ./backend
    image: saikiranasamwar4/taskmanager-backend
```
- **`build: ./backend`**: Build image from Dockerfile in backend folder
- **`image`**: Tag built image with this name/registry for pushing to DockerHub

```yaml
    environment:
      - FLASK_ENV=production
      - SECRET_KEY=7a8f9d2e4c6b1a3e...
      - DATABASE_URL=postgresql://taskmanager:P%40ssw0rd%212026SecureDB@postgres:5432/taskmanager_db
```
- **FLASK_ENV**: Set Flask to production mode (disables debug, etc.)
- **SECRET_KEY**: Encryption key for session data
- **DATABASE_URL**: Connection string to PostgreSQL (note: `postgres` is DNS name of postgres container)

```yaml
    depends_on:
      postgres:
        condition: service_healthy
```
- **Purpose**: Don't start backend until PostgreSQL is healthy
- **`service_healthy`**: Wait for healthcheck to pass, not just container to start

```yaml
  frontend:
    build: ./frontend
    image: saikiranasamwar4/taskmanager-frontend
    ports:
      - "80:80"
```
- **Port 80**: HTTP traffic
- **Multi-stage build**: Smaller frontend image (only final Nginx stage)

```yaml
volumes:
  postgres_data:
  backend_instance:
```
- **Named volumes**: Define volumes referenced by containers
- **Benefit**: Data persists between runs; managed by Docker

---

## Kubernetes Manifests

### 1. Namespace (`k8s/namespace.yaml`)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: taskmanager
```
- **Purpose**: Create isolated Kubernetes namespace for this project
- **Namespace**: Like a virtual cluster - pods in different namespaces can't see each other by default
- **Benefit**: Multiple projects can run on same cluster without conflicts

---

### 2. Secrets (`k8s/secrets.yaml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: taskmanager
type: Opaque
stringData:
  password: "TaskManager2026SecureDB"
```
- **Secret**: Kubernetes object for storing sensitive data (passwords, API keys, etc.)
- **`type: Opaque`**: Raw key-value pairs (most common)
- **`stringData`**: Plain text (Kubernetes automatically base64-encodes)
- **Purpose**: Store database password securely (not in plain YAML or environment variables)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-secret
  namespace: taskmanager
type: Opaque
stringData:
  secret-key: "7a8f9d2e4c6b1a3e..."
  database-url: "postgresql://taskmanager:TaskManager2026SecureDB@postgres:5432/taskmanager_db"
```
- **secret-key**: Flask session encryption key
- **database-url**: Connection string (note: references `postgres` service within cluster)

---

### 3. Persistent Volume Claim (`k8s/postgres-pvc.yaml`)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: taskmanager
spec:
  storageClassName: gp2
```
- **PersistentVolumeClaim (PVC)**: Request for storage from Kubernetes
- **`storageClassName: gp2`**: AWS General Purpose SSD (cost-effective, good performance)

```yaml
  accessModes:
  - ReadWriteOnce
```
- **`ReadWriteOnce`**: Storage can be mounted as read-write by single pod
- **Other modes**: ReadOnlyMany (many pods read), ReadWriteMany (many pods read/write)

```yaml
  resources:
    requests:
      storage: 5Gi
```
- **Purpose**: Request 5GB of storage for database
- **Automatic**: Kubernetes creates corresponding PersistentVolume (actual storage)
- **Persistence**: Data survives pod restarts, redeploys on same node

---

### 4. PostgreSQL Deployment (`k8s/postgres-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: taskmanager
spec:
  replicas: 1
```
- **Deployment**: Kubernetes controller that manages pod replicas
- **`replicas: 1`**: Keep exactly 1 PostgreSQL pod running
- **Why not more?**: StatefulSets are better for databases, but Deployment works for learning

```yaml
  selector:
    matchLabels:
      app: postgres
```
- **Selector**: Deployment finds and manages pods with label `app: postgres`
- **Reconciliation**: Kubernetes continuously ensures 1 pod with this label exists

```yaml
    spec:
      automountServiceAccountToken: false
```
- **Security**: Pod doesn't get default service account token
- **Purpose**: Pod can't access Kubernetes API (least privilege)

```yaml
      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
```
- **Container**: Pod contains this container
- **Port**: Container listens on 5432 (metadata only, doesn't restrict access)

```yaml
        env:
        - name: POSTGRES_USER
          value: taskmanager
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
```
- **Environment variables**: Configure PostgreSQL
- **secretKeyRef**: Load password from Secret instead of hardcoding

```yaml
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - taskmanager
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 6
```
- **Readiness Probe**: Is pod ready to receive traffic?
- **`initialDelaySeconds: 10`**: Wait 10s before first check (give container time to start)
- **`periodSeconds: 5`**: Check every 5 seconds
- **`failureThreshold: 6`**: Mark unhealthy after 6 failed checks (30 seconds total)
- **Effect**: Don't route traffic until database is ready

```yaml
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - taskmanager
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
```
- **Liveness Probe**: Is pod still alive/functional?
- **`initialDelaySeconds: 30`**: Wait longer before first check
- **Effect**: If pod stops responding, Kubernetes automatically restarts it

```yaml
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
          subPath: pgdata
```
- **volumeMounts**: Attach storage to pod's filesystem
- **`subPath: pgdata`**: Use subdirectory of PVC to avoid permission conflicts

```yaml
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```
- **Requests**: Kubernetes reserves this much CPU/memory for pod guarantee
- **Limits**: Pod can use up to this much, then gets throttled/killed
- **CPU units**: 1000m = 1 CPU; 250m = 0.25 CPU
- **Scheduling**: Kubernetes only schedules pod if cluster has space for requests

```yaml
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```
- **Volume source**: Use PVC created earlier for persistent storage

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: taskmanager
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```
- **Service**: Virtual IP that provides stable DNS name + load balancing
- **selector**: Routes traffic to pods with `app: postgres` label
- **`port: 5432`**: Service listens on 5432
- **`targetPort: 5432`**: Forwards to container port 5432
- **`type: ClusterIP`**: Internal only (not exposed outside cluster)
- **DNS**: Other pods access as `postgres.taskmanager.svc.cluster.local:5432`

---

### 5. Backend Deployment (`k8s/backend-deployment.yaml`)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```
- **RollingUpdate**: Replace old pods gradually with new ones (zero downtime)
- **`maxUnavailable: 1`**: During update, allow 1 pod to be down
- **`maxSurge: 1`**: During update, create 1 extra pod (so 2 running temporarily)
- **Process**: Start 2 pods → stop 1 old → confirm new works → stop 1 more old

```yaml
        startupProbe:
          httpGet:
            path: /api/health
            port: 8888
          failureThreshold: 30
          periodSeconds: 5
```
- **Startup Probe**: Is pod still starting up?
- **`failureThreshold: 30`**: Allow up to 30 failures × 5 seconds = 150 seconds to start
- **Purpose**: Give pod time to initialize without restarting prematurely

```yaml
        livenessProbe:
          httpGet:
            path: /api/health
            port: 8888
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
```
- **Liveness Probe**: Is pod still running properly?
- **`httpGet`**: Make HTTP request to health endpoint
- **Effect**: If 3 consecutive checks fail, Kubernetes restarts the pod

```yaml
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 8888
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```
- **Readiness Probe**: Should this pod receive traffic?
- **Path**: `/api/ready` endpoint (different from liveness)
- **Effect**: Service removes unhealthy pods from load balancer

---

### 6. Frontend Deployment (`k8s/frontend-deployment.yaml`)

```yaml
  replicas: 2
```
- **2 replicas**: Run 2 frontend instances (stateless, easy to scale)
- **High Availability**: If 1 pod fails, traffic goes to the other

---

### 7. Ingress (`k8s/ingress.yaml`)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: taskmanager-ingress
  namespace: taskmanager
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```
- **Ingress**: Kubernetes object for external HTTP(S) routing into cluster
- **`rewrite-target: /`**: Rewrite URL paths before forwarding

```yaml
spec:
  ingressClassName: nginx
  rules:
  - host: taskmanager.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 8888
```
- **routing**: Requests to `taskmanager.local/` go to frontend (port 80)
- **API routing**: Requests to `taskmanager.local/api` go to backend (port 8888)
- **pathType: Prefix**: Match all paths starting with specified prefix
- **Effect**: Creates virtual load balancer routing external traffic into cluster

---

## Jenkins CI/CD Pipeline

### `Jenkinsfile`

**Purpose**: Automated build, test, scan, and deploy pipeline

```groovy
pipeline {
    agent any
```
- **agent any**: Run on any available Jenkins node

```groovy
    environment {
        DOCKER_REGISTRY = 'saikiranasamwar4'
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/taskmanager-backend"
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/taskmanager-frontend"
    }
```
- **Environment variables**: Available to all stages
- **DOCKER_REGISTRY**: DockerHub username for pushing images

```groovy
        stage('Git Checkout') {
            steps {
                echo 'Checking out source code from Git...'
                checkout scm
            }
        }
```
- **Purpose**: Pull latest code from Git repository
- **`checkout scm`**: Jenkins automatically detects repo from webhook trigger

```groovy
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        sh '/usr/bin/sonar-scanner ...'
                    }
                }
            }
        }
```
- **Purpose**: Static code analysis for bugs and security issues
- **withCredentials**: Securely load token from Jenkins secrets
- **withSonarQubeEnv**: Use SonarQube environment configuration
- **Quality Gate**: Can fail pipeline if code quality issues found

```groovy
        stage('Docker Build - Backend') {
            steps {
                dir('backend') {
                    sh '''
                        docker build -t saikiranasamwar4/taskmanager-backend:v1.0 .
                        docker tag ... :${BUILD_NUMBER}
                        docker tag ... :latest
                    '''
                }
            }
        }
```
- **Purpose**: Build Docker image from Dockerfile
- **Tagging**: 3 tags:
  - `v1.0`: Version tag (stable)
  - `${BUILD_NUMBER}`: Build number (unique per pipeline run)
  - `latest`: Most recent version

```groovy
        stage('Free Port 5432') {
            steps {
                sh '''
                    echo "Stopping Host PostgreSQL if running..."
                    sudo systemctl stop postgresql || true
                    docker compose down || true
                '''
            }
        }
```
- **Purpose**: Clean up ports/containers before integration tests
- **`|| true`**: Ignore errors (service might not be running)
- **Need**: Port 5432 must be free for test database

```groovy
        stage('Docker Test') {
            steps {
                sh '''
                    docker compose up -d
                    echo "Waiting for containers to start..."
                    sleep 40
                    
                    echo "Testing Backend..."
                    curl -f http://localhost:8888 || exit 1
                '''
            }
        }
```
- **Purpose**: Integration test - spin up full stack locally
- **`docker compose up -d`**: Start in background
- **`sleep 40`**: Wait for services to be ready
- **`curl -f`**: Make HTTP request, fail if status is error
- **Effect**: Validates app runs correctly in containers

```groovy
        stage('Docker Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', ...)]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        
                        docker push saikiranasamwar4/taskmanager-backend:v1.0
                        docker push saikiranasamwar4/taskmanager-backend:${BUILD_NUMBER}
                        docker push saikiranasamwar4/taskmanager-backend:latest
                    '''
                }
            }
        }
```
- **Purpose**: Upload built images to DockerHub registry
- **Credentials**: Load from Jenkins secrets
- **Multiple tags**: Each tag pushed separately (points to same image)
- **Availability**: Images now available for Kubernetes to pull

```groovy
        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        # Configure kubectl
                        aws eks update-kubeconfig --name taskmanager-eks --region us-east-1
                        
                        # Setup namespace
                        kubectl apply -f k8s/namespace.yaml
                        
                        # Update image tags to current build
                        sed -i "s|saikiranasamwar4/taskmanager-backend:v1.0|saikiranasamwar4/taskmanager-backend:${BUILD_NUMBER}|g" k8s/backend-deployment.yaml
                    '''
                }
            }
        }
```
- **`update-kubeconfig`**: Configure kubectl to access EKS cluster
- **`kubectl apply`**: Create Kubernetes resources from YAML files
- **`sed`**: Replace image tag in deployment YAML with current build number
- **Effect**: Updates deployment to use newly built image

```groovy
                        # Deploy everything
                        kubectl apply -f k8s/secrets.yaml
                        kubectl apply -f k8s/postgres-pvc.yaml
                        kubectl apply -f k8s/postgres-deployment.yaml
                        
                        # Wait for PostgreSQL
                        echo "Waiting for PostgreSQL..."
                        kubectl rollout status deployment/postgres -n taskmanager --timeout=300s
```
- **Sequential deployment**: Create secrets, storage, database first
- **`rollout status`**: Wait for deployment to be ready (all pods running and healthy)
- **Dependency chain**: Don't deploy backend until PostgreSQL is ready

```groovy
                        # Rollout new version with health checks
                        kubectl rollout status deployment/backend -n taskmanager --timeout=300s || {
                            echo "=== Backend Debug ==="
                            kubectl get pods -n taskmanager -l app=backend -o wide
                            kubectl describe pods -n taskmanager -l app=backend | tail -40
                            kubectl logs -n taskmanager -l app=backend --tail=20 || true
                            exit 1
                        }
```
- **Rollout status**: Wait for rolling update to complete
- **`|| { ... exit 1 }`**: If rollout fails, show debugging info and fail pipeline
- **Debugging**: Show pod status, descriptions, logs to troubleshoot failures

---

## Terraform Infrastructure

### 1. Provider Configuration (`terraform/provider.tf`)

```hcl
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```
- **`required_version`**: Minimum Terraform CLI version
- **`required_providers`**: Specify AWS provider and version constraints
- **`~> 5.0`**: Allow 5.0 to 5.x but not 6.0 (prevents breaking changes)

```hcl
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```
- **Region**: AWS region for all resources
- **default_tags**: Automatically apply these tags to all resources
- **Benefit**: Easy cost allocation, resource identification

---

### 2. VPC Configuration (`terraform/vpc.tf`)

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true
}
```
- **VPC (Virtual Private Cloud)**: Isolated network for this infrastructure
- **CIDR block**: IP range (e.g., 10.0.0.0/16 = 65,536 addresses)
- **DNS**: Enable DNS queries within VPC

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}
```
- **Internet Gateway**: Enables communication between VPC and internet
- **Attached to VPC**: All traffic to/from internet goes through this

```hcl
resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
```
- **Subnets**: Subdivisions of VPC across availability zones
- **`count`**: Create multiple subnets (high availability across AZs)
- **public**: Has route to internet (for load balancers, NAT gateways)
- **`map_public_ip_on_launch`**: Auto-assign public IPs to instances

```hcl
  tags = {
    "kubernetes.io/role/elb"                        = "1"
    "kubernetes.io/cluster/${var.eks_cluster_name}" = "shared"
  }
```
- **Kubernetes tags**: EKS automatically uses subnets with these tags
- **elb**: Subnet can have load balancers
- **shared**: Subnet belongs to this EKS cluster

```hcl
resource "aws_eip" "nat" {
  domain = "vpc"
}
```
- **Elastic IP**: Static public IP for the NAT gateway
- **Why needed**: NAT gateway needs consistent IP

```hcl
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
}
```
- **NAT Gateway**: Allows private subnet resources to access internet (one-way)
- **Placed in public subnet**: So it can access internet
- **Cost**: Expensive (~$32/month), but necessary for private node security

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
```
- **Route Table**: Rules for how traffic is routed
- **Route**: Send all traffic (0.0.0.0/0) to internet gateway
- **Effect**: Public subnet resources can reach internet directly

---

### 3. EKS Cluster Configuration (`terraform/eks.tf`)

```hcl
resource "aws_iam_role" "eks_cluster" {
  name = "${var.project_name}-eks-cluster-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "eks.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })
}
```
- **IAM Role**: AWS permissions/identity for EKS control plane
- **assume_role_policy**: Who can use this role?
- **`eks.amazonaws.com`**: AWS EKS service is allowed to assume this role

```hcl
resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.eks_cluster.name
}
```
- **Attach policy**: Give EKS control plane required permissions
- **AmazonEKSClusterPolicy**: AWS managed policy (contains standard EKS permissions)
- **Benefit**: Don't need to write custom policy

```hcl
resource "aws_iam_role" "eks_nodes" {
  name = "${var.project_name}-eks-node-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })
}
```
- **IAM Role for nodes**: EC2 instances that run containers
- **`ec2.amazonaws.com`**: EC2 service can assume this role
- **Purpose**: Nodes need permissions to pull Docker images, write logs, etc.

```hcl
resource "aws_eks_cluster" "main" {
  name     = var.eks_cluster_name
  version  = var.eks_cluster_version
  role_arn = aws_iam_role.eks_cluster.arn
  
  vpc_config {
    subnet_ids = concat(
      aws_subnet.public[*].id,
      aws_subnet.private[*].id
    )
    security_group_ids      = [aws_security_group.eks_cluster.id]
    endpoint_private_access = true
    endpoint_public_access  = true
  }
}
```
- **EKS Cluster**: Managed Kubernetes control plane
- **subnets**: EKS control plane distributed across these subnets
- **security_group_ids**: Firewall rules for control plane
- **endpoint_private_access**: Control plane accessible from within VPC
- **endpoint_public_access**: Control plane accessible from internet (enable for debugging)

```hcl
resource "aws_eks_node_group" "workers" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "${var.project_name}-workers"
  node_role_arn   = aws_iam_role.eks_nodes.arn
  subnet_ids      = aws_subnet.private[*].id
  instance_types  = var.eks_node_instance_types
  
  scaling_config {
    desired_size = var.eks_node_desired
    min_size     = var.eks_node_min
    max_size     = var.eks_node_max
  }
  
  update_config {
    max_unavailable = 1
  }
}
```
- **Node Group**: Set of EC2 instances running container workloads
- **instance_types**: EC2 instance type for nodes (e.g., t3.medium)
- **scaling_config**: Auto Scaling Group settings
  - **desired_size**: Target number of nodes (Kubernetes scales to this)
  - **min_size / max_size**: Auto Scaling bounds
- **update_config**: During node updates, allow 1 node down (rolling update)

---

## Monitoring & Observability

### 1. Prometheus Configuration (`monitoring/prometheus-config.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
```
- **ConfigMap**: Kubernetes object for non-sensitive configuration data
- **scrape_interval**: How often to collect metrics from targets (15 seconds)
- **evaluation_interval**: How often to evaluate alert rules

```yaml
    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
          - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```
- **Job**: Named group of scrape targets
- **kubernetes_sd_configs**: Auto-discover targets from Kubernetes API
- **`role: endpoints`**: Find all Kubernetes service endpoints
- **TLS**: Use HTTPS to authenticate with Kubernetes API

```yaml
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
```
- **Pod discovery**: Scrape all pods with `prometheus.io/scrape: true` annotation
- **relabel_configs**: Transformation rules for discovered targets
- **action: keep**: Only keep targets matching regex

```yaml
      - job_name: 'taskmanager-backend'
        static_configs:
          - targets: ['backend.taskmanager.svc.cluster.local:8888']
```
- **Static config**: Hardcoded target (not auto-discovered)
- **backend.taskmanager.svc.cluster.local**: Kubernetes DNS name for backend service
- **:8888**: Prometheus scrapes metrics from backend on this port

---

### 2. Prometheus Deployment (`monitoring/prometheus-deployment.yaml`)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```
- **Separate namespace**: Monitoring stack isolated from application

```yaml
    spec:
      serviceAccountName: prometheus
```
- **ServiceAccount**: Pod's Kubernetes identity (for API access)
- **Prometheus needs access** to Kubernetes API for service discovery

```yaml
      - name: prometheus
        image: prom/prometheus:v2.48.0
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus'
```
- **--config.file**: Path to configuration (mounted from ConfigMap)
- **--storage.tsdb.path**: Where to store time-series database

```yaml
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus
        - name: prometheus-storage
          mountPath: /prometheus
```
- **prometheus-config**: ConfigMap volume (configuration data)
- **prometheus-storage**: PVC volume (persistent time-series data)

```yaml
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
      - name: prometheus-storage
        persistentVolumeClaim:
          claimName: prometheus-pvc
```
- **Volume sources**: ConfigMap for runtime config, PVC for persistent storage

---

### 3. Grafana Deployment (`monitoring/grafana-deployment.yaml`)

```yaml
      - name: grafana
        image: grafana/grafana:10.2.3
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_USER
          value: admin
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: admin123
```
- **Grafana**: Data visualization and dashboarding UI
- **Port 3000**: Web interface
- **Admin credentials**: Default username/password (change in production!)

```yaml
        volumeMounts:
        - name: grafana-datasources
          mountPath: /etc/grafana/provisioning/datasources
        - name: grafana-dashboards-config
          mountPath: /etc/grafana/provisioning/dashboards
```
- **Auto-provisioning**: ConfigMaps can define data sources and dashboards
- **Grafana loads these at startup**: Pre-configured to scrape Prometheus

---

## Configuration Files

### 1. Nginx Configuration (`frontend/nginx.conf`)

```nginx
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index templates/login.html;
```
- **server block**: Virtual server configuration
- **listen 80**: HTTP traffic on port 80
- **root**: Default directory for serving files
- **index**: Default file if directory requested

```nginx
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
```
- **gzip compression**: Compress responses to reduce bandwidth
- **gzip_vary**: Add Vary header (caches understand compression differences)
- **gzip_min_length**: Only compress files larger than 1KB (tiny files not worth it)

```nginx
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
```
- **X-Frame-Options**: Prevent clickjacking (can't embed in iframe)
- **X-Content-Type-Options**: Prevent MIME-type sniffing attacks
- **X-XSS-Protection**: Enable browser XSS protection

```nginx
    location /css/ {
        alias /usr/share/nginx/html/css/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
```
- **location /css/**: Route for CSS requests
- **expires 1y**: Tell browser to cache for 1 year (immutable assets)
- **add_header Cache-Control**: Long-term caching for static files

```nginx
    location /api/ {
        proxy_pass http://backend:8888/api/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```
- **proxy_pass**: Forward requests to backend service
- **proxy_set_header**: Pass client information to backend
- **X-Real-IP / X-Forwarded-For**: Backend sees original client IP (not Nginx IP)

```nginx
    # Main index - redirect to login
    location = / {
        return 301 /login;
    }
    
    # Fallback for other paths
    location / {
        try_files $uri $uri/ /templates/login.html;
    }
```
- **route root**: Redirect to login page
- **try_files**: Try file, then directory, then fallback to login (SPA behavior)
- **effect**: Any non-existent route loads login page (frontend routing handles it)

---

### 2. Secrets (kubernetes/secrets.yaml)

Already covered above in [Kubernetes Manifests](#kubernetes-manifests)

---

## Jenkins RBAC Configuration (`jenkins/jenkins-rbac.yaml`)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
```
- **ServiceAccount**: Kubernetes identity for Jenkins
- **namespace: jenkins**: Jenkins runs in its own namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```
- **ClusterRole**: Define permissions Jenkins needs
- **resources**: What objects Jenkins can interact with
- **verbs**: What actions are allowed
  - **create/delete**: Manage resources
  - **get/list**: Read resources
  - **patch/update**: Modify resources
  - **watch**: Monitor for changes

```yaml
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```
- **apps group**: Kubernetes resource group (Deployments, etc.)
- **Jenkins needs** to update deployments during CI/CD

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: jenkins
```
- **ClusterRoleBinding**: Attach role to ServiceAccount
- **subjects**: Who gets these permissions
- **effect**: Jenkins ServiceAccount now has all permissions defined in ClusterRole

---

## Summary

| File Type | Purpose | Key Concepts |
|-----------|---------|--------------|
| **Dockerfile** | Build container images | Base image, layers, caching, entry point |
| **docker-compose.yml** | Local multi-container setup | Services, volumes, networking, health checks |
| **Kubernetes YAML** | Production orchestration | Deployments, Pods, Services, Storage, Namespaces |
| **Jenkins Pipeline** | CI/CD automation | Build → Test → Scan → Push → Deploy |
| **Terraform** | Infrastructure as Code | VPC, Subnets, EKS, IAM, Auto-Scaling |
| **Monitoring Config** | Observability | Prometheus scraping, Grafana dashboards |
| **Nginx Config** | Reverse proxy & routing | Gzip, caching, security headers, API forwarding |

---

This documentation was created to help understand how all DevOps components work together in this project. Each file plays a specific role in the deployment pipeline from local development to production EKS cluster.

# 🏢 Azka Management Studio — DevOps Implementation

<div align="center">

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)

**A production-grade DevOps pipeline — from code commit to live Kubernetes deployment on AWS**

[🌐 Portfolio](https://Rokkamravi9676.github.io/portfolio) · [💼 LinkedIn](https://www.linkedin.com/in/ravi-rokkam-aa0b9a1b7) · [📧 Contact](mailto:rokkamravi1999@gmail.com)

</div>

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Tech Stack](#-tech-stack)
- [Infrastructure Setup](#-infrastructure-setup)
- [Kubernetes Setup](#-kubernetes-setup)
- [Repository Structure](#-repository-structure)
- [Pipeline Flow](#-pipeline-flow)
- [Security Implementation](#-security-implementation)
- [Key DevOps Concepts Implemented](#-key-devops-concepts-implemented)
- [Production Issues Resolved](#-production-issues-resolved)
- [Author](#-author)

---

## 🚀 Project Overview

**Azka Management Studio** is a full-stack SaaS project management platform built with React + TypeScript (frontend) and FastAPI + Python (backend), deployed on a **self-managed Kubernetes cluster on AWS EC2**. This repository showcases a complete **end-to-end DevOps implementation** including:

- ✅ Containerization with **Docker multi-stage builds**
- ✅ **Self-managed Kubernetes cluster** on AWS EC2 (kubeadm — 1 Master + 2 Workers)
- ✅ Automated CI/CD with **GitHub Actions**
- ✅ **GitOps** deployment workflow with **ArgoCD**
- ✅ **Zero-downtime** rolling deployments on Kubernetes
- ✅ **PostgreSQL** on a dedicated VM with VPC-only network access
- ✅ **Nginx Ingress Controller** for domain-based routing
- ✅ Secure **SSH key-based** team access management across all VMs
- ✅ **AWS Security Groups** hardened with VPC CIDR restrictions

---

## 🏗️ Architecture

```
Developer
    │
    │  git push origin main
    ▼
┌──────────────────┐
│   GitHub Repo    │  ──── Webhook ────►  GitHub Actions
│  (Source + k8s   │                             │
│   manifests)     │                             │
└──────────────────┘                             │
        ▲                                        │  Stage 1: Checkout Code
        │                                        │  Stage 2: Docker Build (Backend)
        │  git push                              │  Stage 3: Docker Build (Frontend)
        │  (image tag update)                    │  Stage 4: Push to Docker Hub
        │                                        │  Stage 5: Update k8s manifests
        └────────────────────────────────────────┘
                                                 │
                                                 ▼
                                      ┌─────────────────┐
                                      │   Docker Hub    │
                                      │ Image Registry  │
                                      └─────────────────┘
                                                 │
                                                 │  Image stored
                                                 ▼
┌──────────────────┐   Detects           ┌──────────────┐
│   GitHub Repo    │ ── manifest ──────► │    ArgoCD    │
│   k8s/manifests  │   change            │    GitOps    │
└──────────────────┘                     └──────────────┘
                                                 │
                                                 │  kubectl apply
                                                 ▼
                              ┌────────────────────────────────┐
                              │     AWS EC2 Kubernetes         │
                              │   Namespace: azka-management   │
                              │                                │
                              │  ┌──────────────────────────┐ │
                              │  │   Backend Deployment     │ │
                              │  │   FastAPI + Python       │ │
                              │  │   NodePort: 30800        │ │
                              │  └──────────────────────────┘ │
                              │  ┌──────────────────────────┐ │
                              │  │   Frontend Deployment    │ │
                              │  │   React + Nginx          │ │
                              │  │   NodePort: 30300        │ │
                              │  └──────────────────────────┘ │
                              │  ┌──────────────────────────┐ │
                              │  │   Nginx Ingress          │ │
                              │  │   Domain Routing         │ │
                              │  └──────────────────────────┘ │
                              └────────────────────────────────┘
                                             │
                              ┌──────────────┴──────────────┐
                              │                             │
                    ┌─────────▼────────┐         ┌─────────▼────────┐
                    │   Worker Node 1  │         │   Worker Node 2  │
                    │  13.233.92.100   │         │  13.234.37.198   │
                    │  Backend Pods    │         │  Frontend Pods   │
                    └──────────────────┘         └──────────────────┘
                                                          │
                                              ┌───────────▼────────────┐
                                              │  PostgreSQL VM         │
                                              │  172.31.32.246         │
                                              │  VPC-only access       │
                                              │  Port 5432 restricted  │
                                              └────────────────────────┘
```

---

## 🔄 CI/CD Pipeline

The GitHub Actions pipeline has **5 automated stages**:

```
┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌─────────────────┐    ┌──────────────┐
│   Stage 1    │    │     Stage 2      │    │    Stage 3       │    │    Stage 4      │    │   Stage 5    │
│   Checkout   │───►│  Build Backend   │───►│ Build Frontend   │───►│ Push Docker Hub │───►│  ArgoCD Sync │
│     Code     │    │  Docker Image    │    │  Docker Image    │    │    Images       │    │  Auto-deploy │
└──────────────┘    └──────────────────┘    └──────────────────┘    └─────────────────┘    └──────────────┘
    GitHub              FastAPI +               React +                Docker Hub            Kubernetes
   git clone            Python                  Nginx                  image push            rolling update
```

### Stage Details

| Stage | Action | Tool |
|-------|--------|------|
| **Checkout** | Pull latest code from GitHub main branch | GitHub Actions |
| **Build Backend** | Docker build FastAPI Python image | Docker |
| **Build Frontend** | Docker build React + Nginx image with env vars | Docker |
| **Push to Registry** | Push both images to Docker Hub | Docker Hub |
| **ArgoCD Sync** | Auto-sync detects manifest change → rolling update | ArgoCD |

---

## 🛠️ Tech Stack

| Category | Technology | Purpose |
|----------|-----------|---------|
| **Frontend** | React 18 + TypeScript + Vite | Web application UI |
| **Backend** | FastAPI + Python 3.9 | REST API server |
| **Database** | PostgreSQL 14 | Persistent data storage |
| **Containerization** | Docker (multi-stage) | Build & package apps |
| **Web Server** | Nginx | Serve static frontend assets |
| **Registry** | Docker Hub | Public image storage |
| **CI** | GitHub Actions | Pipeline automation |
| **CD / GitOps** | ArgoCD | Kubernetes deployment |
| **Orchestration** | Kubernetes (kubeadm) | Container management |
| **Cloud** | AWS EC2 (Ubuntu 22.04) | Virtual machine infrastructure |
| **Networking** | AWS VPC + Security Groups | Network isolation & security |
| **Ingress** | Nginx Ingress Controller | Domain-based traffic routing |
| **Auth** | JWT + bcrypt | Secure authentication |

---

## 🖥️ Infrastructure Setup

### AWS EC2 Instances

| Node | Role | Private IP | Public IP | Instance |
|------|------|-----------|-----------|----------|
| **Master** | K8s Control Plane | 172.31.34.105 | 43.204.25.169 | t3.medium |
| **Worker 1** | K8s Worker (Backend) | 172.31.41.128 | 13.233.92.100 | t3.medium |
| **Worker 2** | K8s Worker (Frontend) | 172.31.47.28 | 13.234.37.198 | t3.medium |
| **PostgreSQL** | Dedicated DB VM | 172.31.32.246 | — | t3.small |

### Kubernetes Cluster Bootstrap

```bash
# On Master Node — initialize cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Setup kubectl for ubuntu user
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install Flannel CNI plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# On Worker Nodes — join cluster
sudo kubeadm join 172.31.34.105:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# Verify cluster
kubectl get nodes
```

### ArgoCD Installation

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose ArgoCD UI
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Get initial admin password
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## ☸️ Kubernetes Setup

### Namespace & Resources

```bash
# Create namespace
kubectl create namespace azka-management

# Apply ConfigMap with environment variables
kubectl apply -f k8s/manifests/backend-config.yaml

# Create Docker Hub pull secret
kubectl create secret docker-registry dockerhub-secret \
  --namespace azka-management \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<username> \
  --docker-password=<password>

# Apply all manifests
kubectl apply -f k8s/manifests/

# Verify deployment
kubectl get pods -n azka-management
kubectl get svc -n azka-management
kubectl get ingress -n azka-management
```

### Kubernetes Resources Created

| Resource | Name | Purpose |
|----------|------|---------|
| Namespace | `azka-management` | Isolates all app resources |
| ConfigMap | `backend-config` | Environment variables (DB URL, CORS, JWT secret) |
| Deployment | `backend` | FastAPI pods with rolling update |
| Deployment | `frontend` | React + Nginx pods with rolling update |
| Service | `backend-service` | NodePort 30800 → backend pods |
| Service | `frontend-service` | NodePort 30300 → frontend pods |
| Ingress | `azka-ingress` | Nginx Ingress for domain routing |

### ConfigMap (backend-config.yaml)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: azka-management
data:
  DATABASE_URL: "postgresql://postgres:password@172.31.32.246:5432/azkamgmtdb"
  CORS_ORIGINS: "*"
  JWT_SECRET_KEY: "your-secret-key"
  ALGORITHM: "HS256"
  ACCESS_TOKEN_EXPIRE_MINUTES: "1440"
```

---

## 📁 Repository Structure

```
AzkaStudio/
├── 📁 backend/                        # FastAPI Python backend
│   ├── 📁 routers/                    # API route handlers
│   │   ├── auth.py                    # Authentication endpoints
│   │   ├── user.py                    # User management CRUD
│   │   ├── project.py                 # Project management
│   │   └── ...
│   ├── 📁 schemas/                    # Pydantic validation models
│   │   └── user.py                    # UserCreate, UserUpdate, UserResponse
│   ├── 📁 auth/
│   │   └── utils.py                   # bcrypt password hashing/verification
│   ├── 📁 models/                     # SQLAlchemy ORM models
│   ├── requirements.txt               # Python dependencies
│   └── Dockerfile                     # Multi-stage backend Docker build
│
├── 📁 frontend/                       # React TypeScript frontend
│   ├── 📁 src/
│   │   ├── 📁 api/                    # Axios API clients
│   │   │   ├── user.ts                # User CRUD API calls
│   │   │   └── backlog.ts             # Backlog API calls
│   │   ├── 📁 pages/                  # Route page components
│   │   ├── 📁 components/             # Reusable UI components
│   │   └── App.tsx                    # Route definitions
│   ├── .env.production                # Production env vars
│   └── Dockerfile                     # Multi-stage frontend Docker build
│
├── 📁 k8s/
│   └── 📁 manifests/                  # Kubernetes YAML manifests
│       ├── backend-deployment.yaml    # Backend deployment + service
│       ├── frontend-deployment.yaml   # Frontend deployment + service
│       ├── backend-config.yaml        # ConfigMap for env vars
│       └── ingress.yaml               # Nginx Ingress routing rules
│
├── 📁 .github/
│   └── 📁 workflows/
│       └── deploy.yml                 # GitHub Actions CI/CD pipeline
│
└── 📄 README.md
```

---

## 🔧 Pipeline Flow

### How a deployment works (step by step)

```
1. Developer pushes code to GitHub main branch
   └── git push origin main

2. GitHub Actions workflow triggered automatically

3. GitHub Actions Stage 1 — Checkout
   └── actions/checkout@v3

4. GitHub Actions Stage 2 — Build & Push Backend
   └── docker build -t azkaadmin/backend-app:latest ./backend
   └── docker push azkaadmin/backend-app:latest

5. GitHub Actions Stage 3 — Build & Push Frontend
   └── docker build \
         --build-arg VITE_API_URL=http://13.233.92.100:30800 \
         -t azkaadmin/frontend-app:latest ./frontend
   └── docker push azkaadmin/frontend-app:latest

6. ArgoCD detects image/manifest change
   └── Polls Git repo every 3 minutes
   └── Applies updated manifests to Kubernetes

7. Kubernetes performs rolling update
   └── New backend pod starts → readiness probe passes
   └── New frontend pod starts → readiness probe passes
   └── Old pods terminate
   └── Zero downtime ✅

8. App live at:
   Frontend → http://13.234.37.198:30300
   Backend  → http://13.233.92.100:30800
   Domain   → http://azkastudio.azkasolutions.com
```

---

## 🔒 Security Implementation

### AWS Security Groups

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | 0.0.0.0/0 | SSH access (key-based auth only) |
| 30300 | TCP | 0.0.0.0/0 | Frontend NodePort |
| 30800 | TCP | 0.0.0.0/0 | Backend NodePort |
| 5432 | TCP | 172.31.0.0/16 | PostgreSQL — VPC only, never public |
| All ICMP | ICMP | sg-041f4398940a2259 | K8s node communication |
| 6443 | TCP | VPC CIDR | Kubernetes API server |

### SSH Key Management

```bash
# Generate team SSH key pair on PostgreSQL VM
ssh-keygen -t rsa -b 4096 -f ~/.ssh/azka_key -N ""

# Add public key to all VMs for azkaadmin user
sudo bash -c 'echo "PUBLIC_KEY" > /home/azkaadmin/.ssh/authorized_keys'
sudo chmod 700 /home/azkaadmin/.ssh
sudo chmod 600 /home/azkaadmin/.ssh/authorized_keys

# Team members connect with private key only
ssh -i ~/.ssh/azka_key azkaadmin@43.204.25.169   # Master
ssh -i ~/.ssh/azka_key azkaadmin@13.233.92.100   # Worker 1
ssh -i ~/.ssh/azka_key azkaadmin@13.234.37.198   # Worker 2
```

### SSH Config (for easy access)

```
Host master
    HostName 43.204.25.169
    User azkaadmin
    IdentityFile ~/.ssh/azka_key

Host worker1
    HostName 13.233.92.100
    User azkaadmin
    IdentityFile ~/.ssh/azka_key

Host worker2
    HostName 13.234.37.198
    User azkaadmin
    IdentityFile ~/.ssh/azka_key
```

Now connect with just: `ssh master` · `ssh worker1` · `ssh worker2`

---

## 🎯 Key DevOps Concepts Implemented

| Concept | Implementation |
|---------|---------------|
| **Infrastructure as Code** | All K8s resources defined in YAML manifests in Git |
| **GitOps** | ArgoCD syncs cluster state from Git — Git is single source of truth |
| **Zero Downtime Deploy** | Rolling update strategy on all deployments |
| **Immutable Infrastructure** | New Docker image per build — never modify running containers |
| **Secrets Management** | Kubernetes ConfigMaps & Secrets — never stored in code |
| **Network Security** | PostgreSQL accessible only within VPC (172.31.0.0/16) |
| **SSH Hardening** | Key-based auth only, no password login, per-user key pairs |
| **Environment Separation** | Build-time env injection via Docker ARGs for frontend |
| **Multi-node Cluster** | Self-managed K8s with kubeadm on AWS EC2 |
| **Ingress Routing** | Nginx Ingress Controller for domain-based traffic routing |

---

## 🐛 Production Issues Resolved

Real-world debugging experience on live production Kubernetes environment:

| Issue | Root Cause | Fix Applied |
|-------|-----------|-------------|
| **Login failure** | Hardcoded `localhost` URLs in frontend build | Updated `VITE_API_URL` build arg in Dockerfile |
| **bcrypt crash** | `passlib[bcrypt]==1.7.4` library bug (>72 bytes) | Replaced with direct `bcrypt==4.0.1` library |
| **500 on user update** | `UserUpdate` schema missing `is_active` field | Added `is_active: Optional[bool]` to Pydantic schema |
| **Department validation error** | `Management` enum value missing from schema | Added `MANAGEMENT = "Management"` to Department enum |
| **CORS blocked requests** | Backend rejecting cross-origin requests | Set `CORS_ORIGINS=*` in ConfigMap |
| **Settings page redirect** | `ProjectProtectedRoute` wrapping Settings route | Removed `ProjectProtectedRoute` from `/settings` route |
| **PostgreSQL auth fail** | Wrong password in `DATABASE_URL` ConfigMap | Updated ConfigMap with correct credentials |
| **Empty department on update** | Frontend sending `""` instead of `null` | Added payload cleanup filter in `updateUser()` |

---

## 👨‍💻 Author

<div align="center">

**Rokkam Ravi**
DevOps Engineer @ Piersoft Technologies
3+ Years Experience

[![Portfolio](https://img.shields.io/badge/Portfolio-000000?style=for-the-badge&logo=github&logoColor=white)](https://Rokkamravi9676.github.io/portfolio)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/ravi-rokkam-aa0b9a1b7)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Rokkamravi9676)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:rokkamravi1999@gmail.com)

</div>

---

<div align="center">
<sub>Built with ❤️ using Docker · Kubernetes · GitHub Actions · ArgoCD · AWS · FastAPI · React · PostgreSQL</sub>
</div>

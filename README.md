# 🧩 EasyCRUD – Kubernetes + EKS + Ingress + RDS Architecture

<div align="center">

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS EKS](https://img.shields.io/badge/AWS%20EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)

</div>

---

## 🚀 Project Overview

**EasyCRUD** is a full-stack CRUD application deployed on **AWS EKS** using a production-style, security-first Kubernetes architecture.

| Layer | Technology |
|-------|-----------|
| Frontend | React (Vite) |
| Backend | Spring Boot |
| Database | AWS RDS (MariaDB) |
| Orchestration | Kubernetes on AWS EKS |
| Ingress | NGINX Ingress Controller |
| Load Balancer | AWS ELB |

> The architecture enforces proper **network isolation** — only the frontend is publicly reachable. Backend and database are fully internal.

---

## 🏗️ Architecture Diagram

```
                        Internet
                           │
                           ▼
              ┌────────────────────────┐
              │   AWS ELB (Public)     │  ← Entry point
              └────────────┬───────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │  NGINX Ingress         │  ← Routing layer
              │  Controller            │
              └──────┬─────────┬───────┘
                     │         │
               path: /       path: /api
                     │         │
                     ▼         ▼
              ┌──────────┐  ┌──────────────┐
              │ Frontend │  │   Backend    │
              │ Service  │  │   Service    │
              │(ClusterIP│  │ (ClusterIP)  │
              └──────────┘  └──────┬───────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │  AWS RDS MariaDB     │
                        │  (Private Subnet)    │
                        └─────────────────────┘
```

---

## 🔐 Security Design

| Component | Exposure | Reason |
|-----------|----------|--------|
| ELB | ✅ Public | Entry point |
| Ingress | 🔒 Internal | Routing layer |
| Frontend Service | 🔒 ClusterIP | Internal only |
| Backend Service | 🔒 ClusterIP | Internal only |
| RDS | 🔒 Private Subnet | No public access |

### Security Controls Implemented

- ✅ No `NodePort` services — all services are `ClusterIP`
- ✅ Backend has **zero** public exposure
- ✅ RDS configured with `Publicly Accessible = No`
- ✅ Ingress-based API routing (no direct backend URL)
- ✅ Environment variables injected securely at build time

---

## 🛠️ Technologies Used

- **AWS EKS** — Managed Kubernetes cluster
- **Docker** — Containerization
- **Kubernetes** — Workload orchestration
- **NGINX Ingress Controller** — Ingress-based routing
- **React + Vite** — Frontend framework
- **Spring Boot** — Backend REST API
- **AWS RDS (MariaDB)** — Managed relational database

---

## 📦 Kubernetes Components

### Deployments
| Name | Description |
|------|-------------|
| `frontend` | React (Vite) application |
| `backend` | Spring Boot REST API |

### Services
| Name | Type | Purpose |
|------|------|---------|
| `frontend-svc` | ClusterIP | Internal frontend access |
| `backend-svc` | ClusterIP | Internal backend access |

### Ingress
Routes all external traffic via NGINX:

| Path | Destination |
|------|-------------|
| `/` | Frontend Service |
| `/api` | Backend Service |

---

## 🌐 API Routing Strategy

Frontend is configured with:

```env
VITE_API_URL=/api
```

Ingress handles routing so the browser never directly contacts the backend:

```yaml
rules:
  - http:
      paths:
        - path: /
          backend: frontend-svc

        - path: /api
          backend: backend-svc
```

> This pattern means the backend URL is **never exposed** to the end user.

---

## 📂 Repository Structure

```
EasyCRUD-Updated/
│
├── frontend/
│   ├── Dockerfile
│   └── src/
│
├── backend/
│   ├── Dockerfile
│   └── src/
│
└── k8s-manifests/
    ├── frontend-deploy.yaml
    ├── backend-deploy.yaml
    ├── services.yaml
    └── ingress.yaml
```

---

## 🚀 Deployment Steps

### 1️⃣ Build and Push Docker Images

```bash
# Frontend
docker build -t <dockerhub-username>/frontend:v1 ./frontend
docker push <dockerhub-username>/frontend:v1

# Backend
docker build -t <dockerhub-username>/backend:v1 ./backend
docker push <dockerhub-username>/backend:v1
```

### 2️⃣ Apply Kubernetes Manifests

```bash
kubectl apply -f k8s-manifests/backend-deploy.yaml
kubectl apply -f k8s-manifests/backend-svc.yaml
kubectl apply -f k8s-manifests/frontend-deploy.yaml
kubectl apply -f k8s-manifests/frontend-svc.yaml
kubectl apply -f k8s-manifests/ingress.yaml
```

### 3️⃣ Access the Application

```bash
kubectl get ingress
```

Copy the ELB hostname from the output and open it in your browser. 🎉

---

## 🔄 Request Flow — Example (User Registration)

```
1. User fills registration form in browser
        │
        ▼
2. Browser sends  POST /api/register
        │
        ▼
3. AWS ELB receives the request
        │
        ▼
4. NGINX Ingress matches path "/api" → routes to backend-svc
        │
        ▼
5. Spring Boot backend processes request
        │
        ▼
6. Backend queries AWS RDS MariaDB
        │
        ▼
7. Response returned → Ingress → ELB → Browser ✅
```

---

## 📈 Scalability

Horizontal scaling is supported out of the box:

```bash
# Scale backend
kubectl scale deployment backend --replicas=3

# Scale frontend
kubectl scale deployment frontend --replicas=3
```

Load balancing is handled automatically by:
- **Kubernetes Service** — distributes traffic across pods
- **AWS ELB** — distributes traffic across nodes

---

## 🧠 Key Learnings

- Proper use of `ClusterIP` vs `LoadBalancer` service types
- Ingress-based routing to avoid backend exposure
- Secure backend and database isolation patterns
- Service → Pod communication model in Kubernetes
- Debugging `503` and `404` errors in Kubernetes
- Environment variable injection in Vite builds (`VITE_*` prefix)

---

## 🔒 Production Improvements (Future Enhancements)

- [ ] Add HTTPS via **cert-manager** + Let's Encrypt
- [ ] Enable **HPA** (Horizontal Pod Autoscaler)
- [ ] Add **liveness & readiness probes**
- [ ] Implement **Network Policies** for zero-trust networking
- [ ] Integrate **CloudWatch** logging & monitoring
- [ ] Add a **CI/CD pipeline** (GitHub Actions / ArgoCD)

---

## 🏁 Final Result

| ✔ Fully Containerized | ✔ Deployed on AWS EKS |
|---|---|
| ✔ Ingress-Based Routing | ✔ Secure RDS Integration |
| ✔ Production-Style Architecture | ✔ Isolated Backend & Database |

---

## 👨‍💻 Author

**mrstark316-lgtm**

---

<div align="center">
  <i>Built with ❤️ using Kubernetes, Spring Boot, React & AWS</i>
</div>

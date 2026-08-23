# 🛒 RoboShop Kubernetes Microservices Project (`k8-roboshop`)

This repository contains Kubernetes manifest files, service networking definitions, configuration decouplings, and operational best practices for deploying the **RoboShop E-Commerce Microservices Application** on a Kubernetes cluster (AWS EKS / Minikube).

---

## 📑 Project Architecture & Components

RoboShop is a distributed 3-tier e-commerce application comprising databases, backend microservices, and a frontend web application:

```mermaid
graph TD
    User([Web Browser]) -->|HTTP :80| Web[Web / Frontend - Nginx]
    Web -->|API Proxy| Cat[Catalogue - Node.js]
    Web -->|API Proxy| UserSvc[User - Node.js]
    Web -->|API Proxy| Cart[Cart - Node.js]
    Web -->|API Proxy| Shipping[Shipping - Java]
    Web -->|API Proxy| Payment[Payment - Python]
    Web -->|API Proxy| Dispatch[Dispatch - Golang]

    Cat -->|TCP :27017| Mongo[(MongoDB)]
    UserSvc -->|TCP :27017| Mongo
    UserSvc -->|TCP :6379| Redis[(Redis)]
    Cart -->|TCP :6379| Redis
    Cart -->|API| Cat
    Shipping -->|TCP :3306| MySQL[(MySQL)]
    Shipping -->|API| Cart
    Payment -->|TCP :5672| RabbitMQ[(RabbitMQ)]
    Payment -->|API| UserSvc
    Dispatch -->|TCP :5672| RabbitMQ
```

---

## 📁 Repository Structure & Manifest Tracker

| Component / Layer | Directory / File | Resource Kind(s) | Port(s) | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Namespace** | [`namespace.yaml`](file:///Users/sriramcharankolla/Desktop/DevOps/k8-roboshop/namespace.yaml) | `Namespace` | N/A | Dedicated `roboshop` namespace for project isolation |
| **Database** | [`mongodb/manifest.yaml`](file:///Users/sriramcharankolla/Desktop/DevOps/k8-roboshop/mongodb/manifest.yaml) | `Deployment`, `Service` | `27017` | NoSQL database for Catalogue & User microservices |
| **Caching** | `redis/` *(Upcoming)* | `Deployment`, `Service` | `6379` | In-memory key-value cache for User sessions & Cart |
| **Relational DB** | `mysql/` *(Upcoming)* | `Deployment`, `Service` | `3306` | Relational database for Shipping transactions & countries |
| **Message Queue** | `rabbitmq/` *(Upcoming)* | `Deployment`, `Service` | `5672` | AMQP Message Broker for asynchronous payment dispatching |
| **Microservice** | `catalogue/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Node.js backend managing product catalog & categories |
| **Microservice** | `user/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Node.js backend managing customer authentication & logins |
| **Microservice** | `cart/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Node.js backend managing items added to user shopping cart |
| **Microservice** | `shipping/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Java Spring Boot microservice calculating shipping rates & orders |
| **Microservice** | `payment/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Python microservice handling payment processing & queues |
| **Microservice** | `dispatch/` *(Upcoming)* | `Deployment`, `Service` | `8080` | Golang worker daemon processing order fulfillment |
| **Frontend** | `frontend/` *(Upcoming)* | `Deployment`, `Service` | `80` | Nginx reverse proxy serving Web UI & routing `/api` calls |

---

## 🚀 Deployment Instructions

### 1. Create Dedicated Project Namespace
Always deploy the `namespace.yaml` first to isolate RoboShop resources from the `default` namespace:

```bash
kubectl apply -f namespace.yaml
kubectl get namespaces
```

---

### 2. Deploy Databases & State Storage

#### MongoDB (`mongodb/manifest.yaml`)
```bash
# Apply MongoDB Deployment & Service inside roboshop namespace
kubectl apply -f mongodb/manifest.yaml

# Verify Deployment, Pod, and Service
kubectl get deployment -n roboshop -l component=mongodb
kubectl get pods -n roboshop -l component=mongodb -o wide
kubectl get svc -n roboshop -l component=mongodb
```

---

## 🛠️ Kubernetes Best Practices Applied in RoboShop

1. **Namespace Isolation:**
   - All components are scoped under `namespace: roboshop` to prevent port collisions, organize RBAC permissions, and enable clear resource tracking.
2. **Declarative Deployments over Bare Pods:**
   - Utilizing `kind: Deployment` ensures automated pod self-healing, scaling, rolling updates with zero downtime, and rollback capabilities.
3. **Consistent Multi-Tier Labeling Strategy:**
   - Standard labels applied across all workloads:
     ```yaml
     labels:
       project: roboshop
       tier: db # or app, frontend
       component: mongodb # or catalogue, user, etc.
     ```
4. **Service Discovery via Kubernetes DNS:**
   - Services expose internal endpoints (e.g. `mongodb:27017`), allowing backend applications to discover dependencies using short service names within the same namespace.
5. **Reliable Transport Protocol (TCP):**
   - Explicit `protocol: TCP` configured for connection-oriented, acknowledged communication between microservices and databases.

---

## 🔍 Verification & Debugging Commands

```bash
# Check all resources inside roboshop namespace
kubectl get all -n roboshop

# Check detailed pod status and events
kubectl describe pod -n roboshop -l component=mongodb

# View live container logs
kubectl logs -n roboshop -l component=mongodb -f

# Verify Service Endpoints mapping
kubectl get endpoints -n roboshop
```

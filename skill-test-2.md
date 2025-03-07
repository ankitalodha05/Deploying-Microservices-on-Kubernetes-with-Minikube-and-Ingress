# Microservices Kubernetes Deployment Assessment

**By:** Ankita Lodha

---

## 📌 Objective

This document provides a step-by-step guide for deploying a **microservices application** on Kubernetes using **Minikube**, ensuring proper **service communication** and **Ingress configuration**.

---

## 1️⃣ Minikube Setup and Initialization

### 🔹 Prerequisites

Microservices application code hosted in GitHub repository

Fork [https://github.com/mohanDevOps-arch/Microservices-Task.git](https://github.com/mohanDevOps-arch/Microservices-Task.git) to [https://github.com/ankitalodha05/Microservices-Task-1.git](https://github.com/ankitalodha05/Microservices-Task-1.git)

Ensure the following tools are installed:

- **Docker Desktop**
- **Minikube**
- **Kubectl**
- **Node.js** (for testing locally)

Clone the GitHub repository:

```sh
git clone https://github.com/ankitalodha05/Microservices-Task-1.git
cd Microservices-Task-1
```

### Step 1: Create Dockerfiles for Microservices

Each microservice (User, Product, Order, Gateway) requires a Dockerfile inside its respective folder.

Navigate to the microservices directory:

```sh
cd Microservices
```

Create a Dockerfile for each microservice:

```dockerfile
# Dockerfile
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the service port
EXPOSE 3000

# Start the service
CMD ["node", "app.js"]
```

Build and tag Docker images:

```sh
docker build -t ankitalodha05/<microservice-name>:latest .
```

Example:

```sh
docker build -t ankitalodha05/user .
docker build -t ankitalodha05/product .
docker build -t ankitalodha05/order .
docker build -t ankitalodha05/gateway .
```

Push images to Docker Hub:

```sh
docker push ankitalodha05/user
docker push ankitalodha05/product
docker push ankitalodha05/order
docker push ankitalodha05/gateway
```

### Step 2: Start Minikube

Run the following command to start **Minikube with Docker**:

```sh
minikube start --driver=docker
```

Verify that Minikube is running:

```sh
kubectl get nodes
```

Enable **Ingress** on Minikube:

```sh
minikube addons enable ingress
```

Verify that the Ingress controller is running:

```sh
kubectl get pods -n ingress-nginx
```

---

## 2️⃣ Deploying Microservices

Each microservice will have:

✔ **Deployment YAML**\
✔ **Service YAML**\
✔ **Correct ports, health probes, environment variables, and labels**

### 🔹 Deployment YAMLs

These files should be in the **`deployments/` directory**.

#### 1️⃣ User Service (`deployments/user-service.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: ankitalodha05/user-service:latest
          ports:
            - containerPort: 3000
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 3
            periodSeconds: 10
```

Repeat similar deployments for **Product Service, Order Service, and Gateway Service**.

---

### 🔹 Service YAMLs

These files should be in the **`services/` directory**.

#### User Service (`services/user-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  type: NodePort
  selector:
    app: user-service
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

Repeat the **same structure** for **Product, Order, and Gateway services**.

---

## 3️⃣ Ingress Configuration

Your Ingress configuration should be in **`ingress/ingress.yaml`**.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: microservices.local
      http:
        paths:
          - path: /users
            pathType: Prefix
            backend:
              service:
                name: user-service
                port:
                  number: 3000
          - path: /products
            pathType: Prefix
            backend:
              service:
                name: product-service
                port:
                  number: 3001
          - path: /orders
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 3002
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gateway-service
                port:
                  number: 3003
```

Apply the manifests:

```sh
kubectl apply -f k8s-manifests/deployments/
kubectl apply -f k8s-manifests/services/
kubectl apply -f k8s-manifests/ingress/
```

## **Verify Deployment**
### **Check Running Pods:**
```bash
kubectl get pods
```
### **Check Services:**
```bash
kubectl get svc
```
### **Check Ingress:**
```bash
kubectl get ingress
```

---
## **Access Services**
1. **Get Minikube IP:**
```bash
minikube ip
```
2. **Add to `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows):**
```
192.168.49.2 microservices.local

## 4️⃣ Testing the Deployment

Run the following:

```sh
curl http://192.168.49.2/users
curl http://192.168.49.2/products
curl http://192.168.49.2/orders
curl http://192.168.49.2/
```

---

## 5️⃣ Troubleshooting Guide

If **Ingress doesn't work**, restart Minikube and reapply configurations:

```sh
minikube delete
minikube start --driver=docker
minikube addons enable ingress
kubectl apply -f k8s-manifests/
```

---

## 📌 Conclusion

🎯 This document provides a **step-by-step guide** to deploying and testing a microservices architecture in **Kubernetes using Minikube**.

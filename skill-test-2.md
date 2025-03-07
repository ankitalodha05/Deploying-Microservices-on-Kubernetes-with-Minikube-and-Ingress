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
-![image](https://github.com/user-attachments/assets/19b6cdfc-9a3c-42c0-9b8e-953aff3e12b6)

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
-![image](https://github.com/user-attachments/assets/c589c4b2-f277-4bb3-9bf6-44427e36f766)
-![image](https://github.com/user-attachments/assets/1a5e87ed-3008-44bc-a906-4bc5810c41ae)
-![image](https://github.com/user-attachments/assets/50edfb8b-6be2-451c-8e3a-4fa18aaefee7)

## **Verify Deployment**
### **Check Running Pods:**
```bash
kubectl get pods
```
-![image](https://github.com/user-attachments/assets/4e305351-ab9e-42bb-9079-78eba66077a7)


### **Check Services:**
```bash
kubectl get svc
```
-
### **Check Ingress:**
```bash
kubectl get ingress
```
-![image](https://github.com/user-attachments/assets/4f3f2783-ab8c-428b-a158-595db80b0c42)

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
-![image](https://github.com/user-attachments/assets/2520e0ee-d950-4b35-be6e-5979e5f60052)

-![image](https://github.com/user-attachments/assets/006b9099-fb0a-42e2-ae38-defcf60496ac)
-![Screenshot 2025-03-06 130714](https://github.com/user-attachments/assets/b2a6962a-17bf-4ecd-996d-1d15b52b12bc)
-![Screenshot 2025-03-06 130728](https://github.com/user-attachments/assets/7035be54-8c8e-4284-822e-bc5d0e5de8ee)

---

## access the server using minikube ssh:

```bash
minikube ssh
curl http://user-service:3000
curl http://product-service:3001
curl http://order-service:3002
curl http://gateway-service:3003
```
-![image](https://github.com/user-attachments/assets/c56e4456-ee0e-4d7c-99f3-87c92de82411)

---
## 5️⃣ Troubleshooting Guide

If **Ingress doesn't work**, restart Minikube and reapply configurations:

```sh

-![image](https://github.com/user-attachments/assets/7e8942c0-e33c-4172-afd4-c90c25ec6247)

-![image](https://github.com/user-attachments/assets/a99ea4f1-0122-4bdf-866e-8e1382e52922)

-![image](https://github.com/user-attachments/assets/bd8bb33e-ee42-4308-95d1-0fc734eb53e5)
-![image](https://github.com/user-attachments/assets/abf39ba6-fbfd-49bc-905d-ec601f528025)
-![image](https://github.com/user-attachments/assets/69989cdf-bf47-47b8-84b3-8bb91f723b85)




minikube delete
minikube start --driver=docker
minikube addons enable ingress
kubectl apply -f k8s-manifests/
```

---

## 📌 Conclusion

🎯 This document provides a **step-by-step guide** to deploying and testing a microservices architecture in **Kubernetes using Minikube**.

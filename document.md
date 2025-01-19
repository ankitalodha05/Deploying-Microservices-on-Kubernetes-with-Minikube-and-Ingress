# Microservices Containerization Assignment

This document provides step-by-step instructions to complete the microservices containerization assignment, including installation of Docker and Docker Compose, granting appropriate permissions, containerizing the services, and deploying them on an EC2 instance.

---

## **Prerequisites**

1. **AWS EC2 Instance**
   - Launch an EC2 instance with Ubuntu 20.04 or similar.
   - Ensure security group rules allow inbound traffic on ports `3000-3003` and SSH (port `22`).

2. **Installed Tools**
   - SSH client (e.g., PuTTY or Terminal).

---

## **Step 1: Install Docker**

Run the following commands to install Docker on the EC2 instance:

```bash
sudo apt update
sudo apt install -y docker.io
```

### Verify Docker Installation:
```bash
docker --version
```
You should see the installed Docker version.

---

## **Step 2: Install Docker Compose**

sudo apt  install docker-compose
```

### Verify Docker Compose Installation:
```bash
docker-compose --version
```

---

## **Step 3: Grant Permissions to Docker Socket**

Grant permissions to the Docker socket so the current user can run Docker commands without `sudo`:

```bash
sudo chmod 777 /var/run/docker.sock
```

## **Step 4: Clone the Repository**

Clone the provided repository containing the microservices:

```bash
git clone <repository-url>
cd Microservices-Task-main
```

Ensure the folder structure looks like this:

```
Microservices-Task-main/
├── user-service/
├── product-service/
├── order-service/
├── gateway-service/
├── docker-compose.yml
└── README.md
```

---

## **Step 5: Write Dockerfiles for Each Service**

Each service requires a `Dockerfile`. Below is an example for `user-service`:

```dockerfile
# Dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

Repeat for the other services, changing only the `EXPOSE` port:
- `product-service`: `EXPOSE 3001`
- `order-service`: `EXPOSE 3002`
- `gateway-service`: `EXPOSE 3003`

---

## **Step 6: Create the Docker Compose File**

The `docker-compose.yml` file orchestrates all services:

```yaml
version: "3.8"
services:
  user-service:
    build:
      context: ./user-service
    ports:
      - "3000:3000"
    networks:
      - microservices-network
  product-service:
    build:
      context: ./product-service
    ports:
      - "3001:3001"
    networks:
      - microservices-network
  order-service:
    build:
      context: ./order-service
    ports:
      - "3002:3002"
    networks:
      - microservices-network
  gateway-service:
    build:
      context: ./gateway-service
    ports:
      - "3003:3003"
    networks:
      - microservices-network
    depends_on:
      - user-service
      - product-service
      - order-service

networks:
  microservices-network:
    driver: bridge
```

---

## **Step 7: Build and Run the Services**

Run the following commands to build and start the containers:

```bash
docker-compose up --build
```

### Verify the Services:
1. Check running containers:
   ```bash
   docker ps
   ```
2. Access services via the public IP of your instance:
   - **User Service:** `http://<public-ip>:3000/users`
   - **Product Service:** `http://<public-ip>:3001/products`
   - **Order Service:** `http://<public-ip>:3002/orders`
   - **Gateway Service:**
     - Users: `http://<public-ip>:3003/api/users`
     - Products: `http://<public-ip>:3003/api/products`
     - Orders: `http://<public-ip>:3003/api/orders`

---

## **Step 8: Troubleshooting**

### Common Issues:
1. **Port Already in Use:**
   Stop any processes using the required ports:
   ```bash
   sudo netstat -tuln | grep <port>
   sudo kill -9 <pid>
   ```

2. **Container Not Starting:**
   Check container logs:
   ```bash
   docker logs <container-name>
   ```

3. **Docker Daemon Not Running:**
   Start the Docker daemon:
   ```bash
   sudo systemctl start docker
   ```

---

## **Screenshots**
1. Docker Compose build and service creation output tested locally.
-![image](https://github.com/user-attachments/assets/e2b3a7cd-1c62-40df-8db8-928a7e5e71a1)

3. `docker ps` showing running containers.
-![image](https://github.com/user-attachments/assets/578489a1-921d-4d3d-b029-27eb0fe34030)


4. Browser screenshots of working endpoints.
-![image](https://github.com/user-attachments/assets/b0df9ee3-e1aa-462a-a8e1-f986a0ff3501)
-![image](https://github.com/user-attachments/assets/94877d01-f52c-49be-8c47-067c10188d5c)
-![image](https://github.com/user-attachments/assets/9523c49d-1150-41ef-9eb9-c9f1fc71ca60)
-![image](https://github.com/user-attachments/assets/e7281b62-ba71-4ace-887d-5f3f27608f12)





---

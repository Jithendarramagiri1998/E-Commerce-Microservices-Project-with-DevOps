# User Service - Microservice Setup

This document provides step-by-step instructions to set up the **User Service** microservice, containerize it with Docker, deploy it using Kubernetes, and integrate it with CI/CD for automated deployment.

---

## 📌 Prerequisites
Ensure you have the following installed:
- **Node.js** (v18+)
- **MongoDB**
- **Docker & Docker Compose**
- **Kubernetes & kubectl**
- **Terraform (for AWS infrastructure)**
- **GitHub Account (for CI/CD)**

---

## 🚀 1. Clone the Repository
```sh
git clone https://github.com/YOUR_GITHUB_USERNAME/user-service.git
cd user-service
```

---

## ⚡ 2. Install Dependencies
```sh
npm install
```

---

## 🔧 3. Environment Configuration
Create a **`.env`** file in the root directory:
```ini
MONGO_URI=mongodb://localhost:27017/userdb
JWT_SECRET=mysecretkey
```

---

## 🏃 4. Run Locally
Start MongoDB and the Node.js service:
```sh
node server.js
```
Check the API:
```sh
curl http://localhost:5001/
```
Expected response:
```
You did a wonderful job! 🎉
```

---

## 📜 5. Server Script (`server.js`)
Create a **`server.js`** file:
```javascript
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');

dotenv.config();

const app = express();
app.use(express.json());

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log("MongoDB Connected"))
    .catch(err => console.log(err));

app.get("/", (req, res) => res.send("You did a wonderful job! 🎉"));

app.listen(5001, () => console.log("User Service running on port 5001"));
```

---

## 🐳 6. Docker Setup

### Create a Dockerfile
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5001
CMD ["node", "server.js"]
```

### Build and Run Docker Container
```sh
docker build -t user-service .
docker run -p 5001:5001 --env-file .env user-service
```

---

## 🐋 7. Docker Compose Setup
Create a **`docker-compose.yaml`** file:
```yaml
version: "3.8"
services:
  user-service:
    build: ./
    ports:
      - "5001:5001"
    depends_on:
      - mongodb
    environment:
      - MONGO_URI=mongodb://mongodb:27017/userdb
  
  mongodb:
    image: mongo
    ports:
      - "27017:27017"
```

Run the services:
```sh
docker-compose up -d
```

---

## ☸️ 8. Kubernetes Deployment

### Create Deployment YAML
Create **`k8s/user-service.yaml`**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 2
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
          image: your-dockerhub-username/user-service:latest
          ports:
            - containerPort: 5001
          env:
            - name: MONGO_URI
              value: "mongodb://mongodb:27017/userdb"
```

### Deploy to Kubernetes
```sh
kubectl apply -f k8s/user-service.yaml
```

Check the running pods:
```sh
kubectl get pods
```

Restart deployment if needed:
```sh
kubectl rollout restart deployment user-service
```

---

## ⚙️ 9. CI/CD with GitHub Actions
Create **`.github/workflows/deploy.yml`**:
```yaml
name: CI/CD Pipeline
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Build Docker Image
        run: |
          docker build -t user-service:latest ./

      - name: Push to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker tag user-service:latest your-dockerhub-username/user-service:latest
          docker push your-dockerhub-username/user-service:latest

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/user-service.yaml
```

Commit and push the code to trigger deployment:
```sh
git add .
git commit -m "Initial setup"
git push origin main
```

---

## 🔍 10. Troubleshooting
### Check Logs in Docker
```sh
docker logs $(docker ps -q --filter "name=user-service")
```

### Check Logs in Kubernetes
```sh
kubectl logs -l app=user-service
```

### Restart the Service
```sh
docker-compose restart user-service
kubectl rollout restart deployment user-service
```

---

## 🎯 Summary
✅ **Microservices** (Node.js + MongoDB)
✅ **Dockerized** (Docker & Docker Compose)
✅ **Orchestrated** (Kubernetes Deployment)
✅ **CI/CD Pipeline** (GitHub Actions)
✅ **Infrastructure as Code** (Terraform AWS)
✅ **Monitoring & Security** (Prometheus, Grafana, Trivy)

---

## 📢 Need Help?
Feel free to reach out via **GitHub Issues** or [LinkedIn](https://www.linkedin.com/in/YOUR_PROFILE/). Happy coding! 🚀


# User Service - Microservice Setup on AWS EKS

This README provides a step-by-step guide to setting up the **User Service** microservice, containerizing it with Docker, deploying it on **AWS EKS (Elastic Kubernetes Service)**, and integrating it with **CI/CD using GitHub Actions**.

---

## 📌 Prerequisites
Ensure you have the following installed and configured:
- **AWS CLI** (`aws configure`)
- **kubectl** (`aws eks update-kubeconfig`)
- **eksctl** (`brew install eksctl` or `choco install eksctl`)
- **Docker & Docker Hub account**
- **GitHub Actions Secrets** (AWS credentials & Docker Hub credentials)

---

## 🚀 1. Clone the Repository
```sh
git clone https://github.com/YOUR_GITHUB_USERNAME/user-service.git
cd user-service
```

---

## 🏗️ 2. Set Up AWS EKS Cluster
Create an EKS cluster with **2 worker nodes**:
```sh
eksctl create cluster --name user-service-cluster --region us-east-1 --nodes 2
```

Verify the cluster is running:
```sh
kubectl get nodes
```

---

## 📜 3. Server Script (`server.js`)
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

## 🔧 4. Environment Configuration
Create a **`.env`** file:
```ini
MONGO_URI=mongodb://localhost:27017/userdb
JWT_SECRET=mysecretkey
```

---

## 🐳 5. Docker Setup

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

## ☸️ 6. Deploy to AWS EKS

### Deploy MongoDB (`k8s/mongodb-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  ports:
  - port: 27017
  selector:
    app: mongodb
```
**Deploy MongoDB:**
```sh
kubectl apply -f k8s/mongodb-deployment.yaml
```

### Deploy User Service (`k8s/user-service.yaml`)
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
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  type: LoadBalancer
  selector:
    app: user-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5001
```
**Deploy the service:**
```sh
kubectl apply -f k8s/user-service.yaml
```

**Check the running services:**
```sh
kubectl get svc
```
Access the service using:
```sh
http://<EXTERNAL-IP>/
```

---

## ⚙️ 7. CI/CD with GitHub Actions
Create **`.github/workflows/deploy.yml`**:
```yaml
name: CI/CD Pipeline
on:
  push:
    branches:
      - main

jobs:
  deploy:
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

      - name: Configure AWS CLI
        run: |
          aws configure set aws_access_key_id ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws configure set aws_secret_access_key ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws configure set region us-east-1

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --region us-east-1 --name user-service-cluster
          kubectl apply -f k8s/user-service.yaml
```

### Steps to Enable CI/CD:
1. **Add GitHub Secrets:**
   - `DOCKER_USERNAME` → Your Docker Hub username
   - `DOCKER_PASSWORD` → Your Docker Hub password
   - `AWS_ACCESS_KEY_ID` → AWS access key
   - `AWS_SECRET_ACCESS_KEY` → AWS secret key

2. **Commit & Push Code:**
```sh
git add .
git commit -m "Deploy User Service to EKS"
git push origin main
```
Your service will be automatically deployed to **AWS EKS** when you push changes! 🎉

---

## 🔍 8. Monitoring & Logs
### Check Running Services
```sh
kubectl get pods
kubectl get svc
```

### Check Logs
```sh
kubectl logs -l app=user-service
```

### Restart Deployment
```sh
kubectl rollout restart deployment user-service
```

---

## 🎯 Summary
✅ **AWS EKS Cluster** created with `eksctl`
✅ **MongoDB & User Service** deployed
✅ **LoadBalancer Service** exposed externally
✅ **CI/CD pipeline with GitHub Actions**
✅ **Logs & Monitoring** enabled

Feel free to reach out via **LinkedIn**


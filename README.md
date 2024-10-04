# 3-Tier Full Stack Application - README

## Project Overview

This repository contains the source code and infrastructure setup for a 3-Tier Full Stack application. The application is built using Node.js for the backend, containerized using Docker, and deployed on AWS EKS via Kubernetes. Jenkins is used for CI/CD automation, with SonarQube and Trivy integrated for code quality and security scanning.

## Contents

- **Node.js Application**: Backend code using Node.js, handling routing, third-party integrations, and database access.
- **Docker**: Containerization of the Node.js application for consistent deployment.
- **Jenkins Pipelines**: Jenkins pipeline automates build, test, scan, and deploy processes.
- **Kubernetes (EKS)**: AWS Elastic Kubernetes Service (EKS) for deployment and scaling.
- **Security & Quality Tools**: SonarQube for code quality analysis and Trivy for vulnerability scanning.

---

## Node.js Application

The application is a Node.js Express server that listens on port `3000`. It integrates with services such as Cloudinary for image storage and Mapbox for geolocation features. These external services' credentials are stored securely in Kubernetes Secrets.

---

## Dockerfile

The Dockerfile uses a multi-stage build to ensure a lightweight and production-optimized image.

```dockerfile
# Stage 1: Build the application
FROM node:14 AS build
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve the application
FROM node:14
WORKDIR /app
COPY --from=build /app /app
EXPOSE 3000
CMD ["npm", "start"]
```

### Building and Running the Docker Image
- **Build the Docker image**:
  ```bash
  docker build -t ashishxojain/camp:latest .
  ```
- **Run the Docker container**:
  ```bash
  docker run -d -p 3000:3000 ashishxojain/camp:latest
  ```

---

## Jenkins Pipeline

The Jenkins pipeline is configured to automate the build, test, security scanning, and deployment of the application. Below is an outline of the pipeline stages and relevant commands
writing and applying jenkins script in pipeline scripts using grrovy language.
```

### Key Pipeline Commands

- **Run Unit Tests**:
  ```bash
  npm test
  ```
- **Run Trivy Scan**:
  ```bash
  trivy fs --format table -o fs-report.html .
  ```
- **Run SonarQube Scan**:
  ```bash
  $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground
  ```
- **Build Docker Image**:
  ```bash
  docker build -t ashishxojain/camp:latest .
  ```
- **Push Docker Image**:
  ```bash
  docker push ashishxojain/camp:latest
  ```
- **Deploy to EKS**:
  ```bash
  kubectl apply -f Manifests/dss.yml
  ```

---

## EKS Cluster Setup

Follow the steps to set up and deploy the application to AWS EKS using the provided `eksScripts`.

### Prerequisites

1. **AWS CLI**: Ensure AWS CLI is installed and configured.
2. **kubectl**: Install `kubectl` to interact with the Kubernetes cluster.
3. **eksctl**: Install `eksctl` for EKS cluster management.

### Steps to Set Up EKS
Using scripts written in eksScripts

1. **Create EKS Cluster**:
   ```bash
   eksctl create cluster --name eks-cluster --region ap-south-1 --nodegroup-name linux-nodes --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed
   ```

2. **Configure Kubernetes Secrets**:
   Apply the secret configuration for secure API key storage:
   ```bash
   kubectl apply -f eksScripts/secrets.yml
   ```

3. **Deploy Application**:
   Deploy the Node.js application to the EKS cluster:
   ```bash
   kubectl apply -f eksScripts/dss.yml
   kubectl apply -f eksScripts/secret.yml
   kubectl apply -f eksScripts/svs.yml
   ```

4. **Assign Roles**:
   Apply the RBAC roles to give appropriate permissions:
   ```bash
   kubectl apply -f eksScripts/role.yml
   ```

5. **Verify the Deployment**:
   Confirm that the deployment was successful:
   ```bash
   kubectl get nodes -n webapps
   kubectl get svc -n webapps
   ```

---

## Kubernetes Manifests

### Secrets Configuration (`secrets.yml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: yelp-camp-secrets
type: Opaque
data:
  CLOUDINARY_CLOUD_NAME: ZGhkcGxvZmZxCg==
  CLOUDINARY_KEY: Njg5NzQ3MzU4MTI4NTQ5Cg==
  CLOUDINARY_SECRET: X2VQWm5IRWxYUzNNVDZFLWVwSDd0MmpoMW40Cg==
  MAPBOX_TOKEN: c2suZXlKMUlqb2lZWE5vYVhOb2FtcGhhVzRpTENKaElqb2lZMjB3ZFdrMlpYVjNNVEEzWlRKeGN6QjRPWGxsZFhab1lTSjkuMkd4NW9pWmQyTjlER3RBSm1yakJfZwo=
  DB_URL: bW9uZ29kYitzcnY6Ly8xMTcyMTIxMDAwNDpQVklPQXNsWkFGSGlxczRPQGNsdXN0ZXIxLnl6ZHlpLm1vbmdvZGIubmV0Lz9yZXRyeVdyaXRlcz10cnVlJnc9bWFqb3JpdHkmYXBwTmFtZT1DbHVzdGVyMQo=
  SECRET: Q2x1c3RlcjEK
```

### Deployment Configuration (`deployment.yml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yelp-camp-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: yelp-camp
  template:
    metadata:
      labels:
        app: yelp-camp
    spec:
      containers:
        - name: yelp-camp-container
          image: ashishjjain/camp:latest
          ports:
            - containerPort: 3000
          env:
            - name: CLOUDINARY_CLOUD_NAME
              valueFrom:
                secretKeyRef:
                  name: y

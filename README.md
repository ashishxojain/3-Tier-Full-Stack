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
## Jenkins manifests
### Jenkins dev pipeline glimpse
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/jenkinsDev.png)
## Jenkins Prod pipeline glimpse
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/jenkinsProd.png)

### EKS Cluster Setup

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
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/eksServices.png)

### Secrets Configuration (`secrets.yml`)
secret keys required to setup database 'Cloudinary' 
used secret key namespaces
- CLOUDINARY CLOUDNAME
- CLOUDINARY KEY
- CLOUDINARY SECRET
- DB URL

## Database Manifests
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/db.png)
                  
                  
### Setting up AWS ec2 
3 active ec2 containers for eks to achieve scalability for large projects
a container for setting up jenkins and aquaTrivy
a container for setting sonarqube 
and a container for node js application
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/EC2Containers.png)
for tools to run on specific ports inbound rules are added
![alt text](https://github.com/ashishxojain/3-Tier-Full-Stack/blob/main/ScreenShots/inboundRules.png)



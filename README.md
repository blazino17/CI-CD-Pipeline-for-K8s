# CI/CD Pipeline for Kubernetes

This project demonstrates setting up a CI/CD pipeline using GitHub Actions to build Docker images, push them to Docker Hub, and deploy them to a local Kubernetes cluster (Minikube).

## Prerequisites

- Docker
- Kubernetes (Minikube)
- kubectl
- GitHub account

## Setup

### Minikube

1. **Install Minikube**:
   - Follow the instructions [here](https://minikube.sigs.k8s.io/docs/start/) to install Minikube on your local machine.

2. **Start Minikube**:
   ```sh
   minikube start
Set kubectl Context:

kubectl config use-context minikube

GitHub Secrets
You need to add the following secrets to your GitHub repository:

DOCKER_HUB_USERNAME: Your Docker Hub username.
DOCKER_HUB_ACCESS_TOKEN: Your Docker Hub access token. You can create one here.
To add secrets in GitHub:

Go to your GitHub repository.
Click on Settings.
Navigate to Secrets and variables > Actions.
Click New repository secret and add the required secrets.
GitHub Actions Workflow
This repository contains a GitHub Actions workflow located at .github/workflows/ci-cd.yaml. The workflow file is configured to:

Checkout the code.
Set up Docker Buildx.
Log in to Docker Hub.
Build and push Docker images to Docker Hub.
Deploy to Minikube using kubectl.
Workflow File

name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_HUB_USERNAME }}/your-repo:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up kubectl
      run: |
        curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
        chmod +x ./kubectl
        sudo mv ./kubectl /usr/local/bin/kubectl

    - name: Deploy to Minikube
      run: |
        kubectl apply -f k8s/deployment.yaml
        
Deployment
The Kubernetes deployment file k8s/deployment.yaml specifies how to deploy the Docker image to the Minikube cluster.

Example Deployment File


apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: your-docker-hub-username/your-repo:latest
        ports:
        - containerPort: 80
        
Running Locally

To run the application locally:

Clone the repository:


git clone https://github.com/your-username/CI-CD-Pipeline-for-K8s.git
cd CI-CD-Pipeline-for-K8s
Start Minikube:


minikube start
Deploy the application to Minikube:


kubectl apply -f k8s/deployment.yaml
Access the application:

minikube service my-app


Contributions are welcome! Please submit a pull request or open an issue to discuss your changes.


This project is licensed under the MIT License - see the LICENSE file for details.


Feel free to customize this `README.md` to better fit your project details and requirements.










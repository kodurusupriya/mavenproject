🚀 CI/CD Real-Time Project – End-to-End DevOps Pipeline

This project demonstrates a complete CI/CD pipeline using modern DevOps tools. It automates source code integration, quality checks, artifact building, Docker image creation, image publishing, and Kubernetes deployment using GitHub, Jenkins, Maven, SonarQube, Docker, Docker Hub, ArgoCD, and Amazon EKS.

📌 Project Overview

This CI/CD pipeline performs:

Continuous Integration

Code checkout from GitHub

Static code analysis via SonarQube

Maven build & artifact packaging

Continuous Deployment

Docker image build

Push image to Docker Hub

Update Kubernetes deployment YAML

Auto-deploy using ArgoCD → Amazon EKS

🛠️ Tools & Technologies Used
Category	Tools
SCM	Git, GitHub
CI Server	Jenkins
Build Tool	Maven
Code Quality	SonarQube
Containerization	Docker
Image Registry	Docker Hub
Orchestration	Kubernetes (Amazon EKS)
GitOps Deployment	ArgoCD
AWS Tools	AWS CLI, Kubectl, Eksctl
🧱 Architecture / Workflow

Developer pushes code → GitHub

Jenkins pipeline triggers

Pipeline runs SonarQube static analysis

Maven builds artifact (JAR/WAR)

Docker builds image

Jenkins pushes image to Docker Hub

Jenkins updates deployment manifest

ArgoCD monitors Git repo and syncs changes

Application deploys automatically to EKS

🚀 Pipeline Stages
1️⃣ Checkout Code

Pulls the latest code from GitHub.

2️⃣ SonarQube Analysis

Runs static analysis for bugs, vulnerabilities, and code smells.

3️⃣ Build Artifact

Uses Maven to generate the build artifact.

4️⃣ Build Docker Image

Creates a Docker image using the project’s Dockerfile.

5️⃣ Push to Docker Hub

Uploads the versioned Docker image to Docker Hub.

6️⃣ Update Deployment File

Replaces image tag in deployment YAML with the new build number.

7️⃣ Deploy via ArgoCD

ArgoCD automatically syncs Kubernetes manifests to EKS.

📄 Jenkinsfile (Pipeline Script)

⚠️ Never store real tokens or passwords in GitHub. Use Jenkins Credentials.

pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/youraccount/yourrepo.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                sh '''
                    mvn sonar:sonar \
                    -Dsonar.host.url=http://<sonarqube-ip>:9000 \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t yourdockerhub/image:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
                    sh 'docker login -u yourusername -p ${dockerhub}'
                }
                sh 'docker push yourdockerhub/image:${BUILD_NUMBER}'
            }
        }

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'githubtoken', variable: 'githubtoken')]) {
                    sh '''
                        git config user.email "your@mail.com"
                        git config user.name "Your Name"

                        sed -i "s/tag:.*/tag:${BUILD_NUMBER}/g" deploymentfiles/deployment.yml

                        git add .
                        git commit -m "Updated image version to ${BUILD_NUMBER}"
                        git push https://${githubtoken}@github.com/youraccount/yourrepo HEAD:main
                    '''
                }
            }
        }
    }
}

🐳 Dockerfile
FROM openjdk:17
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

☸️ Kubernetes Manifests
deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mc-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mc-app
  template:
    metadata:
      labels:
        app: mc-app
    spec:
      containers:
      - name: mc-app
        image: yourdockerhub/image:tag
        ports:
        - containerPort: 8080

service.yml
apiVersion: v1
kind: Service
metadata:
  name: mc-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mc-app

🔧 ArgoCD Setup (Summary)

Install ArgoCD:

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Expose ArgoCD UI:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'


Get password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


Create an ArgoCD Application pointing to your GitHub repo.

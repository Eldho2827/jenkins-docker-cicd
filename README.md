# CI/CD Pipeline with Jenkins & Docker

A complete end-to-end CI/CD pipeline that automatically builds, tests, deploys, and pushes a Node.js application using Jenkins, Docker, and GitHub — hosted on AWS EC2.

---

## 🚀 Project Overview

This project demonstrates a fully automated DevOps pipeline. Every code push to the GitHub repository automatically triggers the Jenkins pipeline, which builds a Docker image, runs a test, deploys the container, and pushes the image to Docker Hub — with zero manual steps.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Jenkins | CI/CD automation server |
| Docker | Containerization |
| GitHub | Source code & webhook trigger |
| AWS EC2 | Cloud server (Ubuntu 22.04) |
| Docker Hub | Container image registry |
| Node.js | Application runtime |

---

## 🏗️ Architecture

```
Developer → GitHub Push
               ↓
         GitHub Webhook
               ↓
         Jenkins Pipeline
        ┌──────────────┐
        │  Clone Repo  │
        │  Build Image │
        │  Test Image  │
        │  Deploy App  │
        │  Push to Hub │
        └──────────────┘
               ↓
     App running on EC2 :3000
     Image pushed to Docker Hub
```

---

## 📁 Project Structure

```
jenkins-docker-cicd/
├── app.js           # Node.js Express application
├── package.json     # Node.js dependencies
├── Dockerfile       # Multi-stage Docker build
└── Jenkinsfile      # Jenkins pipeline definition
```

---

## ⚙️ Pipeline Stages

| Stage | Description |
|-------|-------------|
| Clone | Pulls latest code from GitHub |
| Build | Builds Docker image using multi-stage Dockerfile |
| Test | Verifies the Docker image was built successfully |
| Deploy | Stops old container, runs new one on port 3000 |
| Push | Pushes image to Docker Hub |

---

## 🐳 Dockerfile — Multi-Stage Build

```dockerfile
# Stage 1 — Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Stage 2 — Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .

ENTRYPOINT ["node"]
CMD ["app.js"]
```

**Why multi-stage?** The builder stage installs all dependencies and tools. The production stage copies only the final app — keeping the image small and clean.

---

## 📝 Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "eldho10/cicd-node-app"
    }

    stages {
        stage('Clone') {
            steps { checkout scm }
        }
        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }
        stage('Test') {
            steps {
                sh 'docker images $IMAGE_NAME:latest'
                echo 'Image built successfully!'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker stop node-app || true'
                sh 'docker rm node-app || true'
                sh 'docker run -d --name node-app -p 3000:3000 $IMAGE_NAME:latest'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME:latest'
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
    }
}
```

---

## ☁️ AWS EC2 Setup

- **AMI:** Ubuntu 22.04 LTS
- **Instance Type:** t2.medium
- **Region:** ap-south-1 (Mumbai)

**Security Group Ports:**

| Port | Purpose |
|------|---------|
| 22 | SSH |
| 8080 | Jenkins |
| 3000 | Node.js App |

**Software Installed:**
- Java 21 (OpenJDK) — required by Jenkins
- Jenkins (WAR file via systemd service)
- Docker

---

## 🔗 Webhook Automation

GitHub Webhook triggers the Jenkins pipeline automatically on every `git push`:

1. Go to GitHub repo → **Settings → Webhooks → Add Webhook**
2. Payload URL: `http://<EC2-IP>:8080/github-webhook/`
3. Content type: `application/json`
4. Event: **Just the push event**

In Jenkins → Job → Configure → Build Triggers → enable **GitHub hook trigger for GITScm polling**.

---

## 📦 Docker Hub

Image is available at:
```
docker pull eldho10/cicd-node-app:latest
```

---

## 🖥️ Screenshots

### Jenkins Pipeline — All Stages Passed
![Pipeline Success](screenshots/pipeline-success.png)

### Console Output
![Console Output](screenshots/console-output.png)

### Docker Hub — Image Pushed
![Docker Hub](screenshots/dockerhub.png)

### GitHub Webhook — Delivered
![Webhook](screenshots/webhook.png)

---

## 📌 Key Learnings

- How Jenkins pipelines work using declarative syntax
- Multi-stage Docker builds for lean production images
- Securing credentials in Jenkins using the credentials store
- Automating deployments using GitHub Webhooks
- Running containerized applications on AWS EC2

---

## 👤 Author

**Eldho Sabu**  
AWS DevOps Enthusiast | IT Graduate  
[GitHub](https://github.com/Eldho2827) | [LinkedIn](#)


Here’s the updated **`README.md`**:

---

````markdown
# DevOps Task – CI/CD Pipeline with AWS, Jenkins, GitHub, ECR & CloudWatch

## 📌 Objective
Set up a **CI/CD pipeline** for a Node.js application using **AWS (EC2, ECR, CloudWatch), Jenkins, and GitHub**.  
The pipeline should:
- Build and dockerize the app
- Push image to **Amazon ECR**
- Deploy on **EC2**
- Monitor using **CloudWatch**
- Trigger automatically on GitHub push

---

## 🛠️ Implementation

### 1. Source Code (GitHub)
- Repo created with branching strategy:
  - `main` → Production-ready
  - `dev` → Development
- Added:
  - `Dockerfile` → to containerize app
  - `Jenkinsfile` → to define CI/CD pipeline
  - `.dockerignore` → to optimize build
  - `amazon-cloudwatch-agent.json` → CloudWatch config

---

### 2. Jenkins & Docker (Pre-Installed)
- **EC2 instance** was provisioned with **Jenkins and Docker already installed & running**.  
- Configured Jenkins plugins:
  - **Pipeline**
  - **GitHub Integration**
  - **Docker Pipeline**
  - **Amazon ECR**
- Jenkins connected with GitHub via **Webhook** (push → pipeline trigger).  
- Jenkins job linked to repo with `Jenkinsfile`.

---

### 3. Jenkins Pipeline Stages
1. **Checkout** – Pull code from GitHub  
2. **Build** – Install dependencies & run tests  
3. **Dockerize** – Build Docker image  
4. **Push to ECR** – Authenticate & push image  
5. **Deploy** – Run container on EC2  

Sample `Jenkinsfile`:
```groovy
pipeline {
    agent any
    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "664418956210.dkr.ecr.ap-south-1.amazonaws.com/nodejs-app"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Checkout') {
            steps { git branch: 'main', url: 'https://github.com/<your-repo>.git' }
        }
        stage('Build') {
            steps { sh 'npm install' }
        }
        stage('Dockerize') {
            steps { sh "docker build -t $ECR_REPO:$IMAGE_TAG ." }
        }
        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $ECR_REPO
                docker push $ECR_REPO:$IMAGE_TAG
                """
            }
        }
        stage('Deploy') {
            steps {
                sh """
                docker rm -f nodejs-app || true
                docker run -d -p 3000:3000 --name nodejs-app $ECR_REPO:$IMAGE_TAG
                """
            }
        }
    }
}
````

---

### 4. AWS Setup

* **EC2**: Hosts Jenkins & container
* **ECR**: Stores Docker images
* **IAM Role for EC2** with policies:

  * `AmazonEC2ContainerRegistryFullAccess`
  * `CloudWatchAgentServerPolicy`
  * `CloudWatchLogsFullAccess`

---

### 5. CloudWatch Monitoring

Installed **CloudWatch Agent** on EC2:

```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
```

Start agent with config:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent.json -s
```

* Logs shipped to CloudWatch → `docker-logs` log group
* Metrics collected: CPU, memory, disk

---

## 📊 Architecture Diagram

```
   Developer (GitHub Push)
            │
         Webhook
            │
       ┌──────────────┐
       │   Jenkins    │
       │ (on EC2)     │
       └──────┬───────┘
              │
     Build → Test → Dockerize
              │
       Push to Amazon ECR
              │
       Deploy on EC2 Container
              │
     Logs & Metrics → CloudWatch
```

---

## 🚀 Improvements

* Deploy via ECS/EKS instead of EC2
* Integrate notifications (Slack/Email)

---

## 📸 Proof of Work

Screenshots included in repo:

* Jenkins pipeline success
* Running container on EC2
* ECR with pushed image
* CloudWatch logs



Do you also want me to **embed the screenshots you uploaded into this README** so they appear directly when someone opens the repo?
```

cat > /home/ubuntu/StreamingApp/README.md << 'EOF'
# StreamingApp — MERN DevOps Project

**Student:** Ankit Thakkar  
**GitHub:** https://github.com/ARthakkar92/StreamingApp  
**Fork of:** https://github.com/UnpredictablePrashant/StreamingApp  

---

## Step 1: Version Control with Git

### Fork the Repository
Forked the main repository from UnpredictablePrashant into personal GitHub account.

**Commands used:**
```bash
git clone https://github.com/ARthakkar92/StreamingApp.git
cd StreamingApp
git remote add upstream https://github.com/UnpredictablePrashant/StreamingApp.git
git checkout -b dev
```

**Branch Strategy:**
- `main` — synced with upstream original repo
- `dev` — all development work done here

<!-- SCREENSHOT: Add screenshot of GitHub repo showing "forked from UnpredictablePrashant/StreamingApp" -->
<!-- SCREENSHOT: Add screenshot of GitHub showing 2 branches (main and dev) -->

### Sync with Upstream
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
git checkout dev
git merge main
git push origin dev
```

---

## Step 2: Prepare the MERN Application

### Application Architecture
| Service | Port | Technology |
|---|---|---|
| Frontend | 80 | React.js + Nginx |
| Auth Service | 3001 | Node.js/Express |
| Streaming Service | 3002 | Node.js/Express |
| Admin Service | 3003 | Node.js/Express |
| Chat Service | 3004 | Node.js/Express |
| Database | 27017 | MongoDB |

### Dockerfiles Created
Each service has its own Dockerfile:
- `frontend/Dockerfile` — Multi-stage React build + Nginx
- `backend/authService/Dockerfile`
- `backend/streamingService/Dockerfile`
- `backend/adminService/Dockerfile`
- `backend/chatService/Dockerfile`

### Local Docker Test
```bash
docker compose up --build
docker ps
```

<!-- SCREENSHOT: Add screenshot of "docker ps" showing all 6 containers running -->
<!-- SCREENSHOT: Add screenshot of frontend running at http://54.144.66.69:3000 -->

### ECR Repositories Created
```bash
aws ecr create-repository --repository-name streamingapp-frontend --region us-east-1
aws ecr create-repository --repository-name streamingapp-auth --region us-east-1
aws ecr create-repository --repository-name streamingapp-streaming --region us-east-1
aws ecr create-repository --repository-name streamingapp-admin --region us-east-1
aws ecr create-repository --repository-name streamingapp-chat --region us-east-1
```

### Push Images to ECR
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 640928554403.dkr.ecr.us-east-1.amazonaws.com
docker push 640928554403.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend:latest
docker push 640928554403.dkr.ecr.us-east-1.amazonaws.com/streamingapp-auth:latest
docker push 640928554403.dkr.ecr.us-east-1.amazonaws.com/streamingapp-streaming:latest
docker push 640928554403.dkr.ecr.us-east-1.amazonaws.com/streamingapp-admin:latest
docker push 640928554403.dkr.ecr.us-east-1.amazonaws.com/streamingapp-chat:latest
```

<!-- SCREENSHOT: Add screenshot of AWS ECR console showing all 5 repositories -->
<!-- SCREENSHOT: Add screenshot of ECR showing images pushed with tags -->

---

## Step 3: AWS Environment Setup

### AWS CLI Installation
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### AWS CLI Configuration
```bash
aws configure
# AWS Access Key ID: ***
# AWS Secret Access Key: ***
# Default region: us-east-1
# Default output format: json
```

### Verify Configuration
```bash
aws sts get-caller-identity
```

<!-- SCREENSHOT: Add screenshot of "aws sts get-caller-identity" showing Account ID -->
<!-- SCREENSHOT: Add screenshot of "aws --version" output -->

---

## Step 4: Continuous Integration (CI) using Jenkins

### Jenkins Setup
- Jenkins running on EC2 instance
- URL: https://jenkinsacademics.herovired.com/
- Plugins installed: Git, Pipeline, Docker, AWS Steps

### Jenkins Credentials Added
ANKIT_AWS_ACCESS_KEY_ID     → AWS Access Key ID
ANKIT_AWS_ACCESS_KEY        → AWS Secret Access Key
ANKIT_AWS_ACCOUNT_ID        → 640928554403

### Jenkins Pipeline — Jenkinsfile
```groovy
pipeline {
  agent any
  environment {
    AWS_ACCOUNT_ID = '640928554403'
    AWS_REGION     = 'us-east-1'
    ECR_BASE       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('ECR Login') { ... }
    stage('Build & Push') { ... }
  }
}
```

<!-- SCREENSHOT: Add screenshot of Jenkins pipeline page showing StreamingApp-ARthakkar -->
<!-- SCREENSHOT: Add screenshot of Jenkins showing GREEN successful build -->
<!-- SCREENSHOT: Add screenshot of Jenkins Console Output showing "Finished: SUCCESS" -->

---

## Step 5: Kubernetes Deployment (EKS)

### Tools Installed
```bash
eksctl version   # 0.226.0
kubectl version  # v1.36.1
helm version     # v3.21.0
```

### EKS Cluster Creation
```bash
eksctl create cluster \
  --name streamingapp-cluster \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

<!-- SCREENSHOT: Add screenshot of AWS EKS console showing cluster ACTIVE -->
<!-- SCREENSHOT: Add screenshot of "eksctl get cluster" output -->

### Connect kubectl to EKS
```bash
aws eks update-kubeconfig --region us-east-1 --name streamingapp-cluster
kubectl get nodes
```

<!-- SCREENSHOT: Add screenshot of "kubectl get nodes" showing 2 nodes Ready -->

### Helm Chart Structure

streamingapp-chart/
├── Chart.yaml
├── values.yaml
└── templates/
├── frontend.yaml
├── auth.yaml
├── streaming.yaml
├── admin.yaml
└── chat.yaml

### Deploy with Helm
```bash
helm install streamingapp ./streamingapp-chart
kubectl get pods
kubectl get services
```

<!-- SCREENSHOT: Add screenshot of "kubectl get pods" showing all 5 pods Running -->
<!-- SCREENSHOT: Add screenshot of "kubectl get svc" showing LoadBalancer URL -->

### Frontend Access

URL: http://a2a67053da4f04a9eb9dac77ad7e09fd-2063731993.us-east-1.elb.amazonaws.com

<!-- SCREENSHOT: Add screenshot of frontend running in browser showing login page -->
<!-- SCREENSHOT: Add screenshot of successful login/registration -->

---

## Step 6: Monitoring and Logging

### CloudWatch Log Groups Created
```bash
aws logs create-log-group --log-group-name /streamingapp/frontend --region us-east-1
aws logs create-log-group --log-group-name /streamingapp/auth --region us-east-1
aws logs create-log-group --log-group-name /streamingapp/streaming --region us-east-1
aws logs create-log-group --log-group-name /streamingapp/admin --region us-east-1
aws logs create-log-group --log-group-name /streamingapp/chat --region us-east-1
```

### CloudWatch CPU Alarm
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "StreamingApp-HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --region us-east-1
```

<!-- SCREENSHOT: Add screenshot of CloudWatch Log Groups showing all 5 services -->
<!-- SCREENSHOT: Add screenshot of CloudWatch Alarm "StreamingApp-HighCPU" -->

---

## Step 7: Architecture Diagram

Developer Machine
↓ git push origin dev
GitHub Repository (ARthakkar92/StreamingApp)
↓ webhook trigger
Jenkins CI/CD (jenkinsacademics.herovired.com)
↓ docker build + push
Amazon ECR (5 repositories)
us-east-1
├── streamingapp-frontend
├── streamingapp-auth
├── streamingapp-streaming
├── streamingapp-admin
└── streamingapp-chat
↓ helm upgrade
Amazon EKS Cluster (streamingapp-cluster)
├── frontend pod    → LoadBalancer → Internet
├── auth pod        → ClusterIP
├── streaming pod   → ClusterIP
├── admin pod       → ClusterIP
└── chat pod        → ClusterIP
↓
MongoDB Atlas (Cloud Database)
↓
Amazon CloudWatch (Monitoring + Alerts)

## Step 8: Final Validation

### All Pods Running
```bash
kubectl get pods
```
NAME                         READY   STATUS    RESTARTS   AGE
admin-69c4c5b7d4-r87qd       1/1     Running   0          19h
auth-5cc68cf69f-xzk7b        1/1     Running   0          19h
chat-757c87f96c-9q4t5        1/1     Running   0          19h
frontend-7f45cdcc75-59rfw    1/1     Running   0          19h
streaming-59cbd75c77-ltgfq   1/1     Running   0          19h

### Services Running
```bash
kubectl get svc
```
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)
frontend   LoadBalancer   10.100.62.98    a2a67053da4f04a9eb9dac77ad7e09fd-2063731993.us-east-1.elb.amazonaws.com   80:32519/TCP
auth       ClusterIP      10.100.177.131  <none>                                                                    3001/TCP
streaming  ClusterIP      10.100.166.156  <none>                                                                    3002/TCP
admin      ClusterIP      10.100.38.196   <none>                                                                    3003/TCP
chat       ClusterIP      10.100.51.237   <none>                                                                    3004/TCP

<!-- SCREENSHOT: Add screenshot of kubectl get pods all Running -->
<!-- SCREENSHOT: Add screenshot of kubectl get svc showing LoadBalancer -->
<!-- SCREENSHOT: Add screenshot of application running in browser -->

---

## Repository Structure
StreamingApp/
├── backend/
│   ├── authService/        # Auth microservice (Port 3001)
│   ├── streamingService/   # Streaming microservice (Port 3002)
│   ├── adminService/       # Admin microservice (Port 3003)
│   └── chatService/        # Chat microservice (Port 3004)
├── frontend/               # React.js frontend (Port 80)
├── streamingapp-chart/     # Helm charts
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
├── docs/
│   └── screenshots/        # All proof screenshots
├── docker-compose.yml      # Local development
├── Jenkinsfile             # CI/CD pipeline
└── README.md               # This file

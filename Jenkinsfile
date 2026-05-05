pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ACCOUNT_ID = "324583653988"

        ECR_BACKEND = "324583653988.dkr.ecr.ap-south-1.amazonaws.com/streaming-backend"
        ECR_FRONTEND = "324583653988.dkr.ecr.ap-south-1.amazonaws.com/streaming-frontend"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/fancy1505/StreamingApp.git', branch: 'main'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh 'echo "Current Directory:"'
                sh 'pwd'
                sh 'echo "All Files:"'
                sh 'ls -R'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                echo "Building Backend..."
                if [ -d "backend" ]; then
                    docker build -t backend ./backend
                elif [ -d "server" ]; then
                    docker build -t backend ./server
                else
                    echo "❌ Backend folder not found"
                    exit 1
                fi

                echo "Building Frontend..."
                if [ -d "frontend" ]; then
                    docker build -t frontend ./frontend
                elif [ -d "client" ]; then
                    docker build -t frontend ./client
                else
                    echo "❌ Frontend folder not found"
                    exit 1
                fi
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                docker tag backend:latest $ECR_BACKEND:latest
                docker tag frontend:latest $ECR_FRONTEND:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker push $ECR_BACKEND:latest
                docker push $ECR_FRONTEND:latest
                '''
            }
        }
    }
}
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ACCOUNT_ID = "324583653988"

        ECR_BACKEND = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/streaming-backend"
        ECR_FRONTEND = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/streaming-frontend"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/fancy1505/StreamingApp.git', branch: 'main'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh '''
                echo "Current Directory:"
                pwd
                echo "Files:"
                ls -la
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                cd /var/jenkins_home/workspace/StreamingApp

                echo "Listing files:"
                ls

                docker build -t backend ./backend
                docker build -t frontend ./frontend
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
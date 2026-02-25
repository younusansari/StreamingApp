pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-west-1'
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "latest"
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'docker build -t ${ECR_REGISTRY}/streaming-app/auth-service:${IMAGE_TAG} ./backend/authService'
                }
            }
        }
    }
}
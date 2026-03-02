pipeline {
    agent any

    environment {
        // adjust these to your account/region or parameterise them
        AWS_REGION      = 'us-west-1'
        AWS_ACCOUNT_ID  = '975050024946'               // your account
        ECR_REGISTRY    = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        // service names must match what docker-compose produces
        BACKEND_IMAGE   = "adminService"               // example
        AUTH_IMAGE      = "authService"
        CHAT_IMAGE      = "chatService"
        STREAM_IMAGE    = "streamingService"
        FRONTEND_IMAGE  = "frontend"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building Docker images…'
                sh 'docker-compose build'
            }
        }

        stage('Login to ECR') {
            steps {
                // use the AWS credentials stored in Jenkins
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                          | docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Tag & Push') {
            steps {
                echo 'Tagging and pushing images to ECR…'
                sh '''
                    set -e
                    docker tag ${BACKEND_IMAGE}:latest  $ECR_REGISTRY/${BACKEND_IMAGE}:latest
                    docker push $ECR_REGISTRY/${BACKEND_IMAGE}:latest

                    docker tag ${AUTH_IMAGE}:latest     $ECR_REGISTRY/${AUTH_IMAGE}:latest
                    docker push $ECR_REGISTRY/${AUTH_IMAGE}:latest

                    docker tag ${CHAT_IMAGE}:latest     $ECR_REGISTRY/${CHAT_IMAGE}:latest
                    docker push $ECR_REGISTRY/${CHAT_IMAGE}:latest

                    docker tag ${STREAM_IMAGE}:latest   $ECR_REGISTRY/${STREAM_IMAGE}:latest
                    docker push $ECR_REGISTRY/${STREAM_IMAGE}:latest

                    docker tag ${FRONTEND_IMAGE}:latest  $ECR_REGISTRY/${FRONTEND_IMAGE}:latest
                    docker push $ECR_REGISTRY/${FRONTEND_IMAGE}:latest
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up local images…'
            sh 'docker system prune -f'
        }
    }
}
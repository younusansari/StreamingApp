pipeline {
    agent any

    environment {
        // adjust these to your account/region or parameterise them
        AWS_REGION      = 'us-west-1'
        AWS_ACCOUNT_ID  = '975050024946'               // your account
        ECR_REGISTRY    = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPO = 'yunus/streamingapp'
        IMAGE_TAG = "v1.0.${env.BUILD_ID}"  
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

        // stage('Login to ECR') {
        //     steps {
        //         // use the AWS credentials stored in Jenkins
        //         withCredentials([[
        //             $class: 'AmazonWebServicesCredentialsBinding',
        //             credentialsId: 'aws-creds'
        //         ]]) {
        //             sh '''
        //                 aws ecr get-login-password --region $AWS_REGION \
        //                   | docker login --username AWS --password-stdin $ECR_REGISTRY
        //             '''
        //         }
        //     }
        // }
        stage('Push All Services to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: env.AWS_CREDS_ID
                ]]) {
                    script {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                        def services = ['auth', 'streaming', 'admin', 'chat', 'frontend']                        
                        for (String service : services) {
                            sh "docker push ${ECR_REGISTRY}/${ECR_REPO}:${service}-${IMAGE_TAG}"
                        }
                    }
                }
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
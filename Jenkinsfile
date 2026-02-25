pipeline {
    agent any
    
    environment {
        // AWS Configuration
        AWS_REGION = 'us-west-1'  // Change to your region
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        
        // Image Configuration
        IMAGE_TAG = "${BUILD_NUMBER}"  // Use build number for versioning
        // Alternative: IMAGE_TAG = "${GIT_COMMIT.take(7)}"  // Use git commit hash
        
        // Service names
        AUTH_SERVICE = "streaming-app/auth-service"
        STREAMING_SERVICE = "streaming-app/streaming-service"
        ADMIN_SERVICE = "streaming-app/admin-service"
        CHAT_SERVICE = "streaming-app/chat-service"
        FRONTEND = "streaming-app/frontend"
    }
    
    triggers {
        // Trigger on git push
        githubPush()
        // Or poll every 5 minutes
        // pollSCM('H/5 * * * *')
    }
    
    options {
        // Keep last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Timeout after 1 hour
        timeout(time: 1, unit: 'HOURS')
        // Disable concurrent builds
        disableConcurrentBuilds()
    }
    
    stages {
        
        // ====================================
        // STAGE 1: CHECKOUT CODE
        // ====================================
        stage('Checkout') {
            steps {
                echo "═══════════════════════════════════════"
                echo "📦 STAGE 1: Checking out code..."
                echo "═══════════════════════════════════════"
                checkout scm
                script {
                    sh '''
                        echo "Repository: ${GIT_REPOSITORY}"
                        echo "Branch: ${GIT_BRANCH}"
                        git log -1 --oneline
                    '''
                }
            }
        }
        
        // ====================================
        // STAGE 2: BUILD DOCKER IMAGES
        // ====================================
        stage('Build Docker Images') {
            parallel {
                
                stage('Build Auth Service') {
                    steps {
                        echo "🔨 Building Auth Service..."
                        script {
                            sh '''
                                docker build \
                                    -t ${ECR_REGISTRY}/${AUTH_SERVICE}:${IMAGE_TAG} \
                                    -t ${ECR_REGISTRY}/${AUTH_SERVICE}:latest \
                                    -f ./backend/authService/Dockerfile \
                                    ./backend/authService
                                
                                echo "✓ Auth Service image built"
                                docker images | grep ${AUTH_SERVICE}
                            '''
                        }
                    }
                }
                
                stage('Build Streaming Service') {
                    steps {
                        echo "🔨 Building Streaming Service..."
                        script {
                            sh '''
                                docker build \
                                    -t ${ECR_REGISTRY}/${STREAMING_SERVICE}:${IMAGE_TAG} \
                                    -t ${ECR_REGISTRY}/${STREAMING_SERVICE}:latest \
                                    -f ./backend/streamingService/Dockerfile \
                                    ./backend
                                
                                echo "✓ Streaming Service image built"
                                docker images | grep ${STREAMING_SERVICE}
                            '''
                        }
                    }
                }
                
                stage('Build Admin Service') {
                    steps {
                        echo "🔨 Building Admin Service..."
                        script {
                            sh '''
                                docker build \
                                    -t ${ECR_REGISTRY}/${ADMIN_SERVICE}:${IMAGE_TAG} \
                                    -t ${ECR_REGISTRY}/${ADMIN_SERVICE}:latest \
                                    -f ./backend/adminService/Dockerfile \
                                    ./backend
                                
                                echo "✓ Admin Service image built"
                                docker images | grep ${ADMIN_SERVICE}
                            '''
                        }
                    }
                }
                
                stage('Build Chat Service') {
                    steps {
                        echo "🔨 Building Chat Service..."
                        script {
                            sh '''
                                docker build \
                                    -t ${ECR_REGISTRY}/${CHAT_SERVICE}:${IMAGE_TAG} \
                                    -t ${ECR_REGISTRY}/${CHAT_SERVICE}:latest \
                                    -f ./backend/chatService/Dockerfile \
                                    ./backend
                                
                                echo "✓ Chat Service image built"
                                docker images | grep ${CHAT_SERVICE}
                            '''
                        }
                    }
                }
                
                stage('Build Frontend') {
                    steps {
                        echo "🔨 Building Frontend..."
                        script {
                            sh '''
                                docker build \
                                    -t ${ECR_REGISTRY}/${FRONTEND}:${IMAGE_TAG} \
                                    -t ${ECR_REGISTRY}/${FRONTEND}:latest \
                                    -f ./frontend/Dockerfile \
                                    ./frontend
                                
                                echo "✓ Frontend image built"
                                docker images | grep ${FRONTEND}
                            '''
                        }
                    }
                }
            }
        }
        
        // ====================================
        // STAGE 3: VERIFY IMAGES
        // ====================================
        stage('Verify Images') {
            steps {
                echo "═══════════════════════════════════════"
                echo "✓ STAGE 3: Verifying all images built..."
                echo "═══════════════════════════════════════"
                script {
                    sh '''
                        echo "All built images:"
                        docker images | grep streaming-app
                        
                        echo ""
                        echo "Total images: $(docker images | grep streaming-app | wc -l)"
                    '''
                }
            }
        }
        
        // ====================================
        // STAGE 4: LOGIN TO ECR
        // ====================================
        stage('Login to ECR') {
            steps {
                echo "═══════════════════════════════════════"
                echo "🔐 STAGE 4: Authenticating with ECR..."
                echo "═══════════════════════════════════════"
                script {
                    withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                        sh '''
                            echo "Logging in to ECR..."
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REGISTRY}
                            
                            echo "✓ Successfully logged in to ECR"
                        '''
                    }
                }
            }
        }
        
        // ====================================
        // STAGE 5: PUSH IMAGES TO ECR
        // ====================================
        stage('Push Images to ECR') {
            parallel {
                
                stage('Push Auth Service') {
                    steps {
                        echo "📤 Pushing Auth Service to ECR..."
                        script {
                            sh '''
                                docker push ${ECR_REGISTRY}/${AUTH_SERVICE}:${IMAGE_TAG}
                                docker push ${ECR_REGISTRY}/${AUTH_SERVICE}:latest
                                echo "✓ Auth Service pushed successfully"
                            '''
                        }
                    }
                }
                
                stage('Push Streaming Service') {
                    steps {
                        echo "📤 Pushing Streaming Service to ECR..."
                        script {
                            sh '''
                                docker push ${ECR_REGISTRY}/${STREAMING_SERVICE}:${IMAGE_TAG}
                                docker push ${ECR_REGISTRY}/${STREAMING_SERVICE}:latest
                                echo "✓ Streaming Service pushed successfully"
                            '''
                        }
                    }
                }
                
                stage('Push Admin Service') {
                    steps {
                        echo "📤 Pushing Admin Service to ECR..."
                        script {
                            sh '''
                                docker push ${ECR_REGISTRY}/${ADMIN_SERVICE}:${IMAGE_TAG}
                                docker push ${ECR_REGISTRY}/${ADMIN_SERVICE}:latest
                                echo "✓ Admin Service pushed successfully"
                            '''
                        }
                    }
                }
                
                stage('Push Chat Service') {
                    steps {
                        echo "📤 Pushing Chat Service to ECR..."
                        script {
                            sh '''
                                docker push ${ECR_REGISTRY}/${CHAT_SERVICE}:${IMAGE_TAG}
                                docker push ${ECR_REGISTRY}/${CHAT_SERVICE}:latest
                                echo "✓ Chat Service pushed successfully"
                            '''
                        }
                    }
                }
                
                stage('Push Frontend') {
                    steps {
                        echo "📤 Pushing Frontend to ECR..."
                        script {
                            sh '''
                                docker push ${ECR_REGISTRY}/${FRONTEND}:${IMAGE_TAG}
                                docker push ${ECR_REGISTRY}/${FRONTEND}:latest
                                echo "✓ Frontend pushed successfully"
                            '''
                        }
                    }
                }
            }
        }
        
        // ====================================
        // STAGE 6: VERIFY IN ECR
        // ====================================
        stage('Verify in ECR') {
            steps {
                echo "═══════════════════════════════════════"
                echo "✓ STAGE 6: Verifying images in ECR..."
                echo "═══════════════════════════════════════"
                script {
                    withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                        sh '''
                            echo "Images in ECR:"
                            aws ecr describe-images --repository-name streaming-app/auth-service --region ${AWS_REGION} || true
                            
                            echo ""
                            echo "All repositories:"
                            aws ecr describe-repositories --region ${AWS_REGION} | grep repositoryName || true
                        '''
                    }
                }
            }
        }
    }
    
    // ====================================
    // POST BUILD ACTIONS
    // ====================================
    post {
        success {
            echo "═══════════════════════════════════════"
            echo "✅ PIPELINE SUCCESSFUL!"
            echo "═══════════════════════════════════════"
            script {
                sh '''
                    echo "All images pushed to ECR successfully!"
                    echo "Registry: ${ECR_REGISTRY}"
                    echo "Image Tag: ${IMAGE_TAG}"
                    echo ""
                    echo "Image URIs:"
                    echo "  - ${ECR_REGISTRY}/${AUTH_SERVICE}:${IMAGE_TAG}"
                    echo "  - ${ECR_REGISTRY}/${STREAMING_SERVICE}:${IMAGE_TAG}"
                    echo "  - ${ECR_REGISTRY}/${ADMIN_SERVICE}:${IMAGE_TAG}"
                    echo "  - ${ECR_REGISTRY}/${CHAT_SERVICE}:${IMAGE_TAG}"
                    echo "  - ${ECR_REGISTRY}/${FRONTEND}:${IMAGE_TAG}"
                '''
            }
        }
        
        failure {
            echo "═══════════════════════════════════════"
            echo "❌ PIPELINE FAILED!"
            echo "═══════════════════════════════════════"
            echo "Check logs above for errors"
        }
        
        unstable {
            echo "⚠️ Pipeline is unstable"
        }
        
        always {
            echo "🧹 Cleaning up..."
            script {
                // Optional: Remove dangling images to save space
                sh '''
                    docker image prune -f --filters "dangling=true" || true
                '''
            }
        }
    }
}